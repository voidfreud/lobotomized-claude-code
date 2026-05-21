<!--
name: 'Workflow Script: review-branch'
description: >-
  Bundled review-branch workflow — scopes branch changes then runs
  multi-dimension review and verification
ccVersion: 2.1.146
variables:
  - WORKFLOW_NAME
  - WORKFLOW_DESCRIPTION
  - WORKFLOW_WHEN_TO_USE
  - JSON
  - WORKFLOW_PHASES
-->
export const meta = {
  name: '${WORKFLOW_NAME}',
  description: '${WORKFLOW_DESCRIPTION}',
  whenToUse: '${WORKFLOW_WHEN_TO_USE}',
  phases: ${JSON.stringify(WORKFLOW_PHASES)},
}

// ===== Phase 0: Scope =====
phase("Scope")

const MAX_VERIFY = 25

const SCOPE_SCHEMA = {
  type: 'object',
  required: ['files', 'diffBase', 'stats', 'conventions'],
  properties: {
    files: { type: 'array', items: { type: 'string' } },
    diffBase: { type: 'string' },
    stats: { type: 'string' },
    conventions: { type: 'string' },
  },
}

const scope = await agent(
  "Discover the scope of changes on the current branch.\\n\\n" +
  "1. Find the diff base: try 'git rev-parse origin/main' first. If that fails, " +
  "fall back to 'main'. Use the one that exists as diffBase.\\n" +
  "2. Run 'git diff <diffBase>...HEAD --name-only' for the file list.\\n" +
  "3. Run 'git diff <diffBase>...HEAD --stat' for stats summary.\\n" +
  "4. Read CLAUDE.md at the project root if it exists. Extract a brief summary " +
  "(under 500 words) of coding conventions, patterns, and rules. Empty string if no CLAUDE.md.\\n\\n" +
  "Return the structured scope.",
  { label: 'scope:discover', schema: SCOPE_SCHEMA }
)

if (!scope || scope.files.length === 0) {
  log(scope ? "No changes vs " + scope.diffBase + " — nothing to review." : "Scope discovery skipped.")
  return { report: "No changes to review.", stats: { filesReviewed: 0 } }
}

log(scope.files.length + " files changed vs " + scope.diffBase)

// Shared context header — diff base, files, stats, conventions.
// Each reviewer/verifier runs 'git diff <diffBase>...HEAD' itself.
const CONTEXT_HEADER =
  "<context>\\n" +
  "## Diff base: " + scope.diffBase + "\\n\\n" +
  "## Changed files (" + scope.files.length + ")\\n" +
  scope.files.map(function(f) { return "  - " + f }).join("\\n") + "\\n\\n" +
  "## Stats\\n" + scope.stats + "\\n\\n" +
  (scope.conventions ? "## Conventions (CLAUDE.md)\\n" + scope.conventions + "\\n" : "") +
  "</context>\\n\\n"

// ===== Phase 1+2: Review → Verify (pipeline, no barrier) =====

const DIMENSIONS = [
  {
    key: 'bugs',
    title: 'Bugs',
    focus:
      "correctness issues: null/undefined handling, off-by-one errors, race conditions, " +
      "incorrect error handling, resource leaks (unclosed handles, unbounded caches), " +
      "type confusion, logic errors. Be precise about WHY it's a bug — what input triggers it?",
  },
  {
    key: 'simplicity',
    title: 'Simplicity',
    focus:
      "unnecessary complexity: over-engineering, premature abstraction, unnecessary " +
      "indirection, overly clever code, redundant conditionals, configuration for " +
      "hypothetical needs. Ask: can this be simpler without losing functionality?",
  },
  {
    key: 'architecture',
    title: 'Architecture',
    focus:
      "structural issues: tight coupling, poor cohesion, layering violations, misplaced " +
      "responsibilities, leaky abstractions, modules doing too many things. Is each " +
      "module/function doing one thing well?",
  },
  {
    key: 'dead-code',
    title: 'Dead Code',
    focus:
      "unreachable or unused code: unused exports, unreachable branches, stale feature " +
      "flags, commented-out code, debug leftovers, defensive checks for impossible states. " +
      "Use grep/LSP to verify zero callers before flagging an export as dead.",
  },
  {
    key: 'best-practices',
    title: 'Best Practices',
    focus:
      "hygiene: error handling patterns, type safety (avoid 'any', narrow types), " +
      "async/await correctness (unhandled rejections, missing awaits), resource cleanup, " +
      "naming clarity, avoiding common pitfalls.",
  },
  {
    key: 'patterns',
    title: 'Existing Patterns',
    focus:
      "consistency with existing codebase conventions. Grep for similar existing code and " +
      "compare: does the new code follow the same patterns for state management, error " +
      "handling, file layout, naming? Check CLAUDE.md rules. Flag divergence, not " +
      "stylistic preference.",
  },
]

const FINDING_SCHEMA = {
  type: 'object',
  required: ['file', 'line', 'severity', 'title', 'description', 'suggestion'],
  properties: {
    file: { type: 'string' },
    line: { type: 'number' },
    severity: { enum: ['high', 'medium', 'low'] },
    title: { type: 'string' },
    description: { type: 'string' },
    suggestion: { type: 'string' },
  },
}

const REVIEW_SCHEMA = {
  type: 'object',
  required: ['findings'],
  properties: {
    findings: { type: 'array', items: FINDING_SCHEMA },
  },
}

const VERDICT_SCHEMA = {
  type: 'object',
  required: ['verdict', 'confidence', 'reasoning'],
  properties: {
    verdict: { enum: ['confirmed', 'rejected', 'unclear'] },
    confidence: { enum: ['high', 'medium', 'low'] },
    reasoning: { type: 'string' },
  },
}

function reviewPrompt(dim) {
  return CONTEXT_HEADER +
    "## Role: " + dim.title + " Reviewer\\n\\n" +
    "You are reviewing the code changes on this branch for ONE concern: **" + dim.title + "**.\\n\\n" +
    "Focus: " + dim.focus + "\\n\\n" +
    "Run 'git diff " + scope.diffBase + "...HEAD' to see the changes. Read files in full " +
    "context as needed.\\n\\n" +
    "Ground rules:\\n" +
    "- Only report REAL issues in the " + dim.title + " category. No nitpicks, no style opinions " +
    "unless they violate explicit project conventions.\\n" +
    "- Each finding MUST cite a specific file:line.\\n" +
    "- Be thorough but precise. Ten good findings beat fifty vague ones.\\n" +
    "- Empty findings is a valid result if the code is clean in this dimension.\\n\\n" +
    "Return ONLY findings in the " + dim.title + " category."
}

function verifyPrompt(dim, f) {
  return CONTEXT_HEADER +
    "## Role: Adversarial Verifier\\n\\n" +
    "Your PRIMARY job is to REJECT false positives.\\n\\n" +
    "A reviewer claims this is a " + dim.title + " issue:\\n\\n" +
    "File: " + f.file + ":" + f.line + "\\n" +
    "Severity: " + f.severity + "\\n" +
    "Claim: " + f.title + "\\n\\n" +
    "Their reasoning:\\n" + f.description + "\\n\\n" +
    "Their suggested fix:\\n" + f.suggestion + "\\n\\n" +
    "Your task:\\n" +
    "1. Run 'git diff " + scope.diffBase + "...HEAD -- " + f.file + "' and read " + f.file + " around line " + f.line + ".\\n" +
    "2. Try HARD to find a reason this is NOT a real issue:\\n" +
    "   - Is there code elsewhere that handles this case?\\n" +
    "   - Is this intentional behavior (check comments, git blame, related tests)?\\n" +
    "   - Is the reviewer misreading the code?\\n" +
    "   - Is this theoretically possible but practically irrelevant?\\n" +
    "3. Only confirm if the issue clearly survives scrutiny.\\n\\n" +
    "Reject freely. False positives waste human time. If you're unsure after investigation, " +
    "return 'unclear' — do NOT default to 'confirmed'."
}

// Shared verify budget — sorted-by-severity allocation as each dimension completes.
// First-come gets slots but severity-sorted within each dimension → high sev always passes.
const sevRank = { high: 0, medium: 1, low: 2 }
let verifySlots = MAX_VERIFY
const budgetDropped = []

const results = await pipeline(
  DIMENSIONS,
  // Stage 1: review — each dimension independently reads diff and returns findings.
  async (d) => {
    const review = await agent(reviewPrompt(d), {
      label: 'review:' + d.key,
      phase: 'Review',
      schema: REVIEW_SCHEMA,
    })
    if (!review) return null  // user-skip: propagate null through pipeline
    log(d.title + ": " + review.findings.length + " findings")
    return { dim: d, findings: review.findings }
  },
  // Stage 2: budget + verify — starts AS SOON AS each review finishes (no barrier).
  (r) => {
    if (!r || r.findings.length === 0) return Promise.resolve([])
    // Severity-sort so high-priority findings claim budget first within this dimension.
    const sorted = r.findings.slice().sort(function(a, b) { return sevRank[a.severity] - sevRank[b.severity] })
    const toVerify = sorted.filter(function(f) {
      if (verifySlots <= 0 && sevRank[f.severity] >= 1) {  // high always passes
        budgetDropped.push(Object.assign({}, f, { dimension: r.dim.key }))
        return false
      }
      verifySlots--
      return true
    })
    if (toVerify.length < r.findings.length) {
      log(r.dim.title + ": " + toVerify.length + "/" + r.findings.length + " within budget")
    }
    return parallel(
      toVerify.map((f) => () =>
        agent(verifyPrompt(r.dim, f), {
          label: 'verify:' + f.file.split('/').pop() + ':' + f.line,
          phase: 'Verify',
          schema: VERDICT_SCHEMA,
        }).then((v) => v ? Object.assign({}, f, { dimension: r.dim.key, verdict: v }) : null)
      )
    )
  }
)

// ===== Phase 3: Report =====
phase("Report")

const allFindings = results.flat().filter(Boolean)
const confirmed = allFindings.filter((f) => f.verdict.verdict === 'confirmed')
const unclear = allFindings.filter((f) => f.verdict.verdict === 'unclear')
const rejected = allFindings.filter((f) => f.verdict.verdict === 'rejected')

log(
  confirmed.length + " confirmed, " +
  unclear.length + " unclear, " +
  rejected.length + " rejected"
)

// Sort confirmed for synthesis input (severity desc, then confidence desc).
const confOrder = { high: 0, medium: 1, low: 2 }
confirmed.sort((a, b) => {
  const s = sevRank[a.severity] - sevRank[b.severity]
  if (s !== 0) return s
  return confOrder[a.verdict.confidence] - confOrder[b.verdict.confidence]
})

let report
if (confirmed.length === 0) {
  report =
    "No confirmed issues. " +
    (unclear.length > 0
      ? unclear.length + " unclear findings may warrant a manual look."
      : "Branch looks clean across all six dimensions.")
} else {
  // Synthesis agent: semantic dedup + report. JS dedup by file:line/5 misses
  // cross-dimension dupes (same root cause flagged by both 'bugs' and 'architecture').
  report = await agent(
    "Write a concise code review report from these " + confirmed.length +
    " verified findings. They are sorted by severity but MAY contain semantic duplicates " +
    "— the same underlying issue flagged by multiple review dimensions.\\n\\n" +
    "Findings (JSON):\\n" + JSON.stringify(confirmed, null, 2) + "\\n\\n" +
    "Instructions:\\n" +
    "1. FIRST merge semantic duplicates: findings at the same/nearby file:line with " +
    "overlapping root cause are ONE issue. Combine their descriptions.\\n" +
    "2. Group merged findings by severity (high / medium / low).\\n" +
    "3. For each: file:line, one-line title, brief description, suggested fix. " +
    "Include verifier reasoning if it adds clarity. Note which dimensions flagged it.\\n" +
    "4. End with a summary line: N high, M medium, K low (after dedup).\\n\\n" +
    "Keep it tight. No preamble. Start directly with the findings.",
    { label: 'report:synthesize' }
  )
}

return {
  report,
  stats: {
    filesReviewed: scope.files.length,
    diffBase: scope.diffBase,
    dimensionsReviewed: DIMENSIONS.length,
    totalFindings: allFindings.length,
    confirmed: confirmed.length,
    budgetDropped: budgetDropped.length,
    unclear: unclear.length,
    rejected: rejected.length,
  },
  confirmed: confirmed,
  unclear: unclear.map((f) => ({
    file: f.file,
    line: f.line,
    title: f.title,
    dimension: f.dimension,
    reasoning: f.verdict.reasoning,
  })),
}
