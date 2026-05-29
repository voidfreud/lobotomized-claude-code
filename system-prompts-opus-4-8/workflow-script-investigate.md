<!--
name: 'Workflow Script: investigate'
description: >-
  Bundled investigate workflow — forms and tests hypotheses about a problem
  mechanism
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

const TASK = (typeof args === 'string' && args.trim()) ? args.trim() : ''
if (!TASK) return { error: 'No incident description provided. Pass the incident, error, or question as args.' }

const GATHER_SCHEMA = {
  type: 'object', required: ['timeline', 'evidence', 'scope'],
  properties: {
    timeline: { type: 'string', description: 'What happened, in order' },
    evidence: { type: 'array', items: { type: 'string' }, description: 'Concrete observations with file:line or log refs' },
    scope: { type: 'string', description: 'What is affected and what is not' },
    reproSteps: { type: 'string' },
  },
}
const HYPOTHESIS_SCHEMA = {
  type: 'object', required: ['hypothesis', 'mechanism', 'predicts'],
  properties: {
    hypothesis: { type: 'string', description: 'One-sentence root cause claim' },
    mechanism: { type: 'string', description: 'How this cause produces the observed symptom' },
    predicts: { type: 'array', items: { type: 'string' }, description: 'Testable predictions if this is true' },
    suspectCode: { type: 'string', description: 'file:line if applicable' },
  },
}
const VERDICT_SCHEMA = {
  type: 'object', required: ['refuted', 'evidence'],
  properties: { refuted: { type: 'boolean' }, evidence: { type: 'string' } },
}
const REPORT_SCHEMA = {
  type: 'object', required: ['summary', 'rootCause', 'suggestedFix', 'nextSteps'],
  properties: {
    summary: { type: 'string' },
    rootCause: { type: 'string' },
    suggestedFix: { type: 'string' },
    nextSteps: { type: 'array', items: { type: 'string' } },
    confidence: { enum: ['high', 'medium', 'low'] },
  },
}

// ═══ Phase 1: Gather ═══
phase('Gather')

const gather = await agent(
  "Gather evidence for this investigation. Collect facts only; don't theorize yet.\\n\\n" +
  "## Incident\\n" + TASK + "\\n\\n" +
  "## Instructions\\n" +
  "1. Read referenced logs, traces, error messages, and files. Quote exact lines with their source.\\n" +
  "2. Establish a timeline: what happened first, what followed.\\n" +
  "3. Establish scope: what is broken, what still works, when it started.\\n" +
  "4. If reproducible, note the minimal repro steps.\\n\\n" +
  "Stick to observations, not conclusions.",
  { label: 'gather', schema: GATHER_SCHEMA }
)
if (!gather) return { error: 'Gather step skipped.' }
log('Gathered ' + gather.evidence.length + ' pieces of evidence')

const EVIDENCE_BLOCK =
  "## Incident\\n" + TASK + "\\n\\n" +
  "## Timeline\\n" + gather.timeline + "\\n\\n" +
  "## Scope\\n" + gather.scope + "\\n\\n" +
  "## Evidence\\n" + gather.evidence.map((e, i) => (i + 1) + ". " + e).join("\\n") + "\\n" +
  (gather.reproSteps ? "\\n## Repro\\n" + gather.reproSteps + "\\n" : "")

// ═══ Phase 2: Hypothesize (3 parallel) ═══
phase('Hypothesize')

const ANGLES = [
  { key: 'recent-change', lens: 'Assume a recent code or config change caused this. Check git log, recent deploys, flag flips.' },
  { key: 'data-edge-case', lens: 'Assume the code is fine and a particular input, state, or environment value triggered a latent edge case.' },
  { key: 'infra-timing', lens: 'Assume a race, timeout, resource limit, dependency outage, or ordering issue — not the application logic itself.' },
]

const hypotheses = await parallel(ANGLES.map(a => () =>
  agent(
    EVIDENCE_BLOCK + "\\n## Your angle: " + a.key + "\\n" + a.lens + "\\n\\n" +
    "## Instructions\\n" +
    "Propose one concrete root-cause hypothesis from this angle. Read the relevant code. " +
    "Explain the mechanism: how this cause produces every observation in the evidence list. " +
    "List 2-3 testable predictions that hold if and only if this hypothesis is true.",
    { label: 'hypothesis:' + a.key, phase: 'Hypothesize', schema: HYPOTHESIS_SCHEMA }
  )
))
const hyps = hypotheses.map((h, i) => h ? { ...h, angle: ANGLES[i].key } : null).filter(Boolean)
if (hyps.length === 0) return { error: 'No hypotheses generated.', gather }
log(hyps.length + ' hypotheses: ' + hyps.map(h => h.angle).join(', '))

// ═══ Phase 3: Verify (adversarial refutation) ═══
phase('Verify')

const verdicts = await parallel(hyps.map(h => () =>
  agent(
    EVIDENCE_BLOCK + "\\n## Hypothesis under test (" + h.angle + ")\\n" +
    h.hypothesis + "\\n\\nMechanism: " + h.mechanism + "\\n" +
    "Predictions: " + h.predicts.join('; ') + "\\n" +
    (h.suspectCode ? "Suspect: " + h.suspectCode + "\\n" : "") + "\\n" +
    "## Instructions\\n" +
    "Try to refute this hypothesis. Check each prediction against the codebase and evidence, " +
    "and look for evidence the hypothesis can't explain.\\n" +
    "Set refuted=true if any prediction fails or any evidence contradicts the mechanism; " +
    "refuted=false only if every prediction checks out and nothing contradicts it.\\n" +
    "Evidence must cite file:line or a specific observation number.",
    { label: 'refute:' + h.angle, phase: 'Verify', schema: VERDICT_SCHEMA }
  ).then(v => ({ ...h, verdict: v }))
))

const survived = verdicts.filter(v => v && v.verdict && !v.verdict.refuted)
const refuted = verdicts.filter(v => v && v.verdict && v.verdict.refuted)
log('Verify: ' + survived.length + ' survived, ' + refuted.length + ' refuted')

// ═══ Phase 4: Report ═══
phase('Report')

const survivedBlock = survived.length > 0
  ? survived.map(h => "### " + h.hypothesis + " (" + h.angle + ")\\n" +
      "Mechanism: " + h.mechanism + "\\n" +
      (h.suspectCode ? "Suspect: " + h.suspectCode + "\\n" : "") +
      "Verifier evidence: " + h.verdict.evidence + "\\n").join("\\n")
  : "(none survived — all hypotheses refuted)"

const refutedBlock = refuted.map(h =>
  "- " + h.hypothesis + " (" + h.angle + ") — refuted: " + h.verdict.evidence).join("\\n")

const report = await agent(
  "Write the root-cause report.\\n\\n" + EVIDENCE_BLOCK + "\\n" +
  "## Surviving hypotheses (" + survived.length + ")\\n" + survivedBlock + "\\n\\n" +
  "## Refuted hypotheses (" + refuted.length + ")\\n" + (refutedBlock || "(none)") + "\\n\\n" +
  "## Instructions\\n" +
  (survived.length === 1
    ? "One hypothesis survived — that is the root cause. "
    : survived.length > 1
    ? "Multiple hypotheses survived — pick the one that best explains all evidence, or note they may compound. "
    : "No hypothesis survived — synthesize the most likely cause from what was learned during refutation, with low confidence. ") +
  "Write: a 2-3 sentence summary, the root cause, a concrete suggested fix (file:line where " +
  "possible), confidence level, and next steps (further verification, monitoring, follow-ups).",
  { label: 'report', schema: REPORT_SCHEMA }
)
if (!report) return { error: 'Report step skipped.', gather, survived, refuted }

return {
  summary: report.summary,
  rootCause: report.rootCause,
  suggestedFix: report.suggestedFix,
  confidence: report.confidence || (survived.length === 1 ? 'high' : survived.length > 1 ? 'medium' : 'low'),
  nextSteps: report.nextSteps,
  hypotheses: {
    survived: survived.map(h => ({ angle: h.angle, hypothesis: h.hypothesis, suspect: h.suspectCode || null })),
    refuted: refuted.map(h => ({ angle: h.angle, hypothesis: h.hypothesis, reason: h.verdict.evidence })),
  },
  evidence: gather.evidence,
}
