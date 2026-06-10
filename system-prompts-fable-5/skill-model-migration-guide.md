<!--
name: 'Skill: Model migration guide'
description: >-
  Step-by-step instructions for migrating existing code to newer Claude models,
  covering breaking changes, deprecated parameters, per-SDK syntax,
  prompt-behavior shifts, and migration checklists
ccVersion: 2.1.158
-->
# Model Migration Guide

> If you arrived via \`/claude-api migrate\`: this is the right file. Execute the steps below in order. Start with Step 0 (confirm scope) before touching any file.

How to move existing code to newer Claude models. Covers breaking changes, deprecated parameters, and drop-in replacements for retired models.

For the latest authoritative version (with code samples in every supported language), WebFetch the **Migration Guide** URL from \`shared/live-sources.md\`. Use this file as the consolidated, skill-resident reference; fall back to the live docs when a model launch or breaking change may have shifted the picture.

This file is large. Use the section names below to jump (or \`Grep\` for the heading text). Read Step 0 and Step 1 first — they apply to every migration. Then read only the per-target section for the model you are migrating to.

| Section | When you need it |
|---|---|
| Step 0: Confirm the migration scope | Always — before any edits |
| Step 1: Classify each file | Always — decides whether to swap, add-alongside, or skip |
| Per-SDK Syntax Reference | Translate the Python examples to TypeScript / Go / Ruby / Java / C# / PHP |
| Destination Models / Retired Model Replacements | Picking a target model |
| Breaking Changes by Source Model | Migrating to Opus 4.6 / Sonnet 4.6 |
| Migrating to Opus 4.7 | Migrating to Opus 4.7 (breaking changes, silent defaults, behavioral shifts) |
| Opus 4.7 Migration Checklist | Required vs optional items for 4.7, tagged \`[BLOCKS]\` / \`[TUNE]\` |
| Migrating to Opus 4.8 | Migrating to Opus 4.8 (no new breaking changes; mid-session system prompts; behavioral re-tuning) |
| Opus 4.8 Migration Checklist | Required vs optional items for 4.8, tagged \`[BLOCKS]\` / \`[TUNE]\` |
| Verify the Migration | After edits — runtime spot-check |

**TL;DR:** Change the model ID string. If you used \`budget_tokens\`, switch to \`thinking: {type: "adaptive"}\`. Assistant prefills 400 on both Opus 4.6 and Sonnet 4.6 — switch to a prefill replacement (most often \`output_config.format\`; see Breaking Changes by Source Model). Moving Sonnet 4.5 → Sonnet 4.6: set \`effort\` explicitly — 4.6 defaults to \`high\`. Remove the \`effort-2025-11-24\` and \`fine-grained-tool-streaming-2025-05-14\` beta headers (GA on 4.6); remove \`interleaved-thinking-2025-05-14\` once on adaptive thinking (keep it only while using the transitional \`budget_tokens\` escape hatch). Then drop from \`client.beta.messages.create\` back to \`client.messages.create\`. Dial back aggressive "CRITICAL: YOU MUST" tool instructions; 4.6 follows the system prompt much more closely.

---

## Step 0: Confirm the migration scope

Before any Write, Edit, or MultiEdit call, confirm the scope. If the request does not name a single file, a specific directory, or an explicit file list, ask first — do not start editing. Even imperative-sounding requests like "migrate my codebase", "move my project to X", "upgrade to Sonnet 4.6", or bare "migrate to Opus 4.7" leave the scope ambiguous. Phrases like "my project", "my code", "the whole thing", "everywhere", or "across the repo" tell you *what* to do, not *where* — they are not directive about scope.

Offer the common scopes explicitly and wait for the answer before touching any file:

1. The entire working directory
2. A specific subdirectory (e.g. \`src/\`, \`app/\`, \`services/billing/\`)
3. A specific file or a list of files

Surface this as a single clarifying question. Proceed without asking only when the scope is already unambiguous — the user named an exact file ("migrate \`extract.py\` to Sonnet 4.6"), pointed at a directory ("migrate everything under \`services/billing/\` to Opus 4.6"), listed files ("update \`a.py\` and \`b.py\`"), or already answered the scope question earlier. If you can answer "which files will this change touch?" with a precise list from the prompt alone, proceed. If not, ask.

**Worked example.** *"Move my project to Opus 4.6. I want adaptive thinking everywhere it makes sense."* — \`everywhere\` makes the intent clear (every call site *within scope*) but the scope itself is undefined. Do not start editing. Respond:

> Before I start editing, can you confirm the scope? I can migrate:
> 1. Every \`.py\` file in the working directory
> 2. Just the files under \`src/\` (production code)
> 3. A specific subdirectory or list of files you name
>
> Which one?

Then wait. Same for *"Migrate to Opus 4.7"* and bare *"Help me upgrade to Sonnet 4.6"*.

**Sizing the scope question (large repos).** Before asking, get a per-directory count so the user can pick concretely:

\`\`\`sh
rg -l "<old-model-id>" --type-not md | cut -d/ -f1 | sort | uniq -c | sort -rn
\`\`\`

Present the breakdown (e.g. *"Found 217 references across 3 directories: api/ (130), api-go/ (62), routing/ (25). Which to migrate?"*). Confirm \`git status\` is clean before surveying — unexpected modifications mean a concurrent process; stop and investigate.

---

## Step 1: Classify each file

Not every file containing the old model ID is a **caller** of the API. Classify each file before editing — the right action differs:

| # | Bucket | What it looks like | Action |
|---|---|---|---|
| 1 | **Calls the API/SDK** | \`client.messages.create(model=…)\`, \`anthropic.Anthropic()\`, request payloads | Swap the model ID **and** apply the breaking-change checklist for the target version. |
| 2 | **Defines or serves the model** | Model registries, OpenAPI specs, routing/queue configs, model-policy enums, generated catalogs | The old entry stays (the model is still served). Ask whether to (a) add the new model alongside, (b) leave alone, or (c) retire the old model — don't blind-replace. If you can't ask, default to (a) and flag it — replacing would de-register a model still in production. |
| 3 | **References the ID as an opaque string** | UI fallback constants, capability-gate substring checks, generic test fixtures, label parsers, env defaults | Usually swap the string and verify any parser/regex/substring match handles the new ID — but check the sub-cases below first. |
| 4 | **Suffixed variant ID** | \`claude-<model>-<suffix>\` like \`-fast\`, \`-1024k\`, \`-200k\`, \`[1m]\`, dated snapshots | Deployment/routing identifiers, not the public model ID. Don't assume a new-model equivalent exists. Verify in the registry first; if absent, leave the string and flag it. |

**Bucket 3 sub-cases — before swapping a string reference, check:**

- **Capability gate** (e.g. \`if 'opus-4-6' in model_id:\` enables a feature) → add the new ID alongside, don't replace. The old model is still served and still has the capability, so replacing would silently disable the feature for old-model traffic. If no old-model traffic hits this gate (single-caller codebase fully migrating), replacing is fine; if unsure, add alongside.
- **Registry-assert test** (e.g. \`assert "claude-X" in supported_models\`, \`test_X_has_N_clusters\`) → add an assertion for the new model alongside; keep the old one. Heuristic: multiple model versions in a list → registry test; one model in a struct compared only to itself → generic fixture.
- **Frozen / generated snapshot** → regenerate, don't hand-edit.
- **Coupled to a definer** (e.g. an integration test passing model authorization via a shared \`conftest\` seed list, or asserting on a billing-tier / rate-limit-group enum or a generated SKU/pricing catalog) → verify the definer has a new-model entry first. If not, add a seed entry (reusing the nearest existing tier as a placeholder); if you can't do that confidently, ask. Don't skip the test — swapping without populating the definer fails at runtime.

When migrating tests specifically: breaking parameters (\`temperature\`, \`top_p\`, \`budget_tokens\`) are usually absent — fixtures rarely set sampling params on placeholder models. The breaking-change scan is still required, but expect mostly clean results.

**Find intentionally-flagged sync points first.** Many codebases tag spots that must change at every model launch with markers like \`MODEL LAUNCH\`, \`KEEP IN SYNC\`, \`@model-update\`. Grep for the repo's convention *before* the broad model-ID grep — those markers point at the load-bearing changes.

---

## Per-SDK Syntax Reference

Code examples here are Python. The same fields exist in every official Anthropic SDK — Stainless generates all 7 from the same OpenAPI spec, so JSON field names map 1:1 with only case-convention differences. Use the rows below to translate.

> Verify type and method names against the SDK source before writing them into customer code. WebFetch the relevant repository from the SDK source-code table in \`shared/live-sources.md\` (one row per SDK) and confirm the exact symbol — especially for typed SDKs (Go, Java, C#) where union/builder names can differ from the JSON shape. Don't guess type names not in the table below or in \`<lang>/claude-api/README.md\`.

<!-- The rows below were verified against each SDK's \`synced/model-launch-april\` branch. -->

### \`thinking\` — \`budget_tokens\` → adaptive

| SDK | Before | After |
|---|---|---|
| Python | \`thinking={"type": "enabled", "budget_tokens": N}\` | \`thinking={"type": "adaptive"}\` |
| TypeScript | \`thinking: { type: 'enabled', budget_tokens: N }\` | \`thinking: { type: 'adaptive' }\` |
| Go | \`Thinking: anthropic.ThinkingConfigParamOfEnabled(N)\` | \`Thinking: anthropic.ThinkingConfigParamUnion{OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{}}\` |
| Ruby | \`thinking: { type: "enabled", budget_tokens: N }\` | \`thinking: { type: "adaptive" }\` |
| Java | \`.thinking(ThinkingConfigEnabled.builder().budgetTokens(N).build())\` | \`.thinking(ThinkingConfigAdaptive.builder().build())\` |
| C# | \`Thinking = new ThinkingConfigEnabled { BudgetTokens = N }\` | \`Thinking = new ThinkingConfigAdaptive()\` |
| PHP | \`thinking: ['type' => 'enabled', 'budget_tokens' => N]\` | \`thinking: ['type' => 'adaptive']\` |

### Sampling parameters — \`temperature\` / \`top_p\` / \`top_k\`

(Remove the field entirely on Opus 4.7; on Claude 4.x keep at most one of \`temperature\` or \`top_p\`.)

| SDK | Field(s) to remove |
|---|---|
| Python | \`temperature=…\`, \`top_p=…\`, \`top_k=…\` |
| TypeScript | \`temperature: …\`, \`top_p: …\`, \`top_k: …\` |
| Go | \`Temperature: anthropic.Float(…)\`, \`TopP: anthropic.Float(…)\`, \`TopK: anthropic.Int(…)\` |
| Ruby | \`temperature: …\`, \`top_p: …\`, \`top_k: …\` |
| Java | \`.temperature(…)\`, \`.topP(…)\`, \`.topK(…)\` |
| C# | \`Temperature = …\`, \`TopP = …\`, \`TopK = …\` |
| PHP | \`temperature: …\`, \`topP: …\`, \`topK: …\` |

### Prefill replacement — structured outputs via \`output_config.format\`

| SDK | Remove (last assistant turn) | Add |
|---|---|---|
| Python | \`{"role": "assistant", "content": "…"}\` | \`output_config={"format": {"type": "json_schema", "schema": SCHEMA}}\` |
| TypeScript | \`{ role: 'assistant', content: '…' }\` | \`output_config: { format: { type: 'json_schema', schema: SCHEMA } }\` |
| Go | trailing \`anthropic.MessageParam{Role: "assistant", …}\` | \`OutputConfig: anthropic.OutputConfigParam{Format: anthropic.JSONOutputFormatParam{…}}\` |
| Ruby | \`{ role: "assistant", content: "…" }\` | \`output_config: { format: { type: "json_schema", schema: SCHEMA } }\` |
| Java | trailing \`Message.builder().role(ASSISTANT)…\` | \`.outputConfig(OutputConfig.builder().format(JsonOutputFormat.builder()…build()).build())\` |
| C# | trailing \`new Message { Role = "assistant", … }\` | \`OutputConfig = new OutputConfig { Format = new JsonOutputFormat { … } }\` |
| PHP | trailing \`['role' => 'assistant', 'content' => '…']\` | \`outputConfig: ['format' => ['type' => 'json_schema', 'schema' => $SCHEMA]]\` |

### \`thinking.display\` — opt back into summarized reasoning (Opus 4.7)

| SDK | Add |
|---|---|
| Python | \`thinking={"type": "adaptive", "display": "summarized"}\` |
| TypeScript | \`thinking: { type: 'adaptive', display: 'summarized' }\` |
| Go | \`Thinking: anthropic.ThinkingConfigParamUnion{OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized}}\` |
| Ruby | \`thinking: { type: "adaptive", display: "summarized" }\` (or \`display_:\` when constructing the model class directly) |
| Java | \`.thinking(ThinkingConfigAdaptive.builder().display(ThinkingConfigAdaptive.Display.SUMMARIZED).build())\` |
| C# | \`Thinking = new ThinkingConfigAdaptive { Display = Display.Summarized }\` |
| PHP | \`thinking: ['type' => 'adaptive', 'display' => 'summarized']\` |

For any field not in these tables, the JSON key in the Python example translates directly: \`snake_case\` for Python/TypeScript/Ruby, \`camelCase\` named args for PHP, \`PascalCase\` struct fields for Go/C#, \`camelCase\` builder methods for Java.

---

## Explain every change you make

Migration edits often look arbitrary to a user who hasn't read the release notes — a removed \`temperature\`, a deleted prefill, a rewritten system-prompt sentence. For each edit, tell the user what you changed and why, tied to the specific API or behavioral change that motivates it. Do this in your summary as you work, not only at the end.

Be especially explicit about **system-prompt edits** — prompt-tuning changes are judgment calls, not hard API requirements. For any prompt edit:

- Quote the before and after text.
- State the behavioral shift that motivates it (e.g. *"Opus 4.7 calibrates response length to task complexity, so I added an explicit length instruction"*, or *"4.6 follows instructions more literally, so 'CRITICAL: YOU MUST use the search tool' will now overtrigger — softened to 'Use the search tool when…'"*).
- Distinguish **optional tuning** (tone, length, subagent guidance) from code edits **required to avoid a 400** (sampling params, \`budget_tokens\`, prefills). Don't present an optional prompt change as mandatory.

If applying several prompt-tuning edits at once, offer them as a short list the user can accept or decline item-by-item rather than silently rewriting their system prompt.

---

## Before You Migrate

1. **Confirm the target model ID.** Use only the exact strings from \`shared/models.md\` — don't append date suffixes to aliases (\`claude-opus-4-6\`, not \`claude-opus-4-6-20251101\`). Guessing an ID will 404.
2. **Check which features your code uses:**
   - \`thinking: {type: "enabled", budget_tokens: N}\` → migrate to adaptive thinking on Opus 4.6 / Sonnet 4.6 (still functional but deprecated)
   - Assistant-turn prefills (\`messages\` ending with \`role: "assistant"\`) → must change on Opus 4.6 / Sonnet 4.6 (returns 400)
   - \`output_format\` parameter on \`messages.create()\` → must change on all models (deprecated API-wide)
   - \`max_tokens > ~16000\` → must stream on any model. When streaming, Sonnet 4.6 / Haiku 4.5 cap at 64K and Opus 4.6 caps at 128K
   - Beta headers \`effort-2025-11-24\`, \`fine-grained-tool-streaming-2025-05-14\`, \`interleaved-thinking-2025-05-14\` → GA on 4.6, remove them and switch from \`client.beta.messages.create\` to \`client.messages.create\`
   - Moving Sonnet 4.5 → Sonnet 4.6 with no \`effort\` set → 4.6 defaults to \`high\`, changing latency/cost
   - System prompts with \`CRITICAL\`, \`MUST\`, \`If in doubt, use X\` language → likely to overtrigger on 4.6 (see Prompt-Behavior Changes)
   - Coming from 3.x / 4.0 / 4.1: also check sampling params (\`temperature\` + \`top_p\`), tool versions (\`text_editor_20250728\`), \`refusal\` + \`model_context_window_exceeded\` stop reasons, trailing-newline tool-param handling
3. **Test on a single request first.** Run one call against the new model, inspect the response, then roll out.

---

## Destination Models (recommended targets)

| If you're on…                         | Migrate to         | Why                                               |
| ------------------------------------- | ------------------ | ------------------------------------------------- |
| Opus 4.7                              | \`claude-opus-4-8\`  | Most capable model; same API surface as 4.7 (no new breaking changes) — mostly prompt re-tuning; see Migrating to Opus 4.8 |
| Opus 4.6                              | \`claude-opus-4-8\`  | Apply the Opus 4.7 breaking changes, then the 4.8 re-tuning |
| Opus 4.0 / 4.1 / 4.5 / Opus 3         | \`claude-opus-4-8\`  | Apply 4.6 → 4.7 → 4.8 in order (adaptive thinking, drop sampling params, then re-tune) |
| Sonnet 4.0 / 4.5 / 3.7 / 3.5          | \`claude-sonnet-4-6\`| Best speed / intelligence balance; adaptive thinking; 64K output |
| Haiku 3 / 3.5                         | \`claude-haiku-4-5\` | Fastest and most cost-effective                   |

Default to the latest Opus for the caller's tier unless they explicitly chose otherwise. The Opus migrations layer: on Opus 4.6 or older, apply each version's section in order up to your target (e.g. 4.5 → 4.8 means the 4.6, 4.7, and 4.8 sections in sequence). A 4.7 → 4.8 move has no new breaking changes — see Migrating to Opus 4.8.

---

## Retired Model Replacements

These models return 404 — update immediately:

| Retired model                 | Retired       | Drop-in replacement  |
| ----------------------------- | ------------- | -------------------- |
| \`claude-3-7-sonnet-20250219\`  | Feb 19, 2026  | \`claude-sonnet-4-6\`  |
| \`claude-3-5-haiku-20241022\`   | Feb 19, 2026  | \`claude-haiku-4-5\`   |
| \`claude-3-opus-20240229\`      | Jan 5, 2026   | \`claude-opus-4-8\`    |
| \`claude-3-5-sonnet-20241022\`  | Oct 28, 2025  | \`claude-sonnet-4-6\`  |
| \`claude-3-5-sonnet-20240620\`  | Oct 28, 2025  | \`claude-sonnet-4-6\`  |
| \`claude-3-sonnet-20240229\`    | Jul 21, 2025  | \`claude-sonnet-4-6\`  |
| \`claude-2.1\`, \`claude-2.0\`    | Jul 21, 2025  | \`claude-sonnet-4-6\`  |

## Deprecated Models (retiring soon)

| Model                         | Retires       | Replacement          |
| ----------------------------- | ------------- | -------------------- |
| \`claude-3-haiku-20240307\`     | Apr 19, 2026  | \`claude-haiku-4-5\`   |
| \`claude-opus-4-20250514\`      | June 15, 2026 | \`claude-opus-4-8\`    |
| \`claude-sonnet-4-20250514\`    | June 15, 2026 | \`claude-sonnet-4-6\`  |

---

## Breaking Changes by Source Model

### Migrating from Sonnet 4.5 to Sonnet 4.6 (effort default change)

Sonnet 4.5 had no \`effort\` parameter; Sonnet 4.6 defaults to \`high\`. Just switching the model string can give noticeably higher latency and token usage. Set \`effort\` explicitly.

**Recommended starting points:**

| Workload                                          | Start at       | Notes                                                                                                    |
| ------------------------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------- |
| Chat, classification, content generation          | \`low\`          | With \`thinking: {"type": "disabled"}\` you'll see similar or better performance vs. Sonnet 4.5 no-thinking |
| Most applications (balanced)                      | \`medium\`       | The default sweet spot for quality vs. cost                                                              |
| Agentic coding, tool-heavy workflows              | \`medium\`       | Pair with adaptive thinking and a generous \`max_tokens\` (up to 64K with streaming — Sonnet 4.6's ceiling) |
| Autonomous multi-step agents, long-horizon loops  | \`high\`         | Scale down to \`medium\` if latency/tokens become a concern                                                 |
| Computer-use agents                               | \`high\` + adaptive | Sonnet 4.6's best computer-use accuracy is on adaptive + high                                          |

For non-thinking chat workloads specifically:

\`\`\`python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8192,
    thinking={"type": "disabled"},
    output_config={"effort": "low"},
    messages=[{"role": "user", "content": "..."}],
)
\`\`\`

**When to use Opus 4.6 instead:** hardest and longest-horizon problems — large code migrations, deep research, extended autonomous work. Sonnet 4.6 wins on fast turnaround and cost.

### Migrating to Opus 4.6 / Sonnet 4.6 (from any older model)

**1. Manual extended thinking is deprecated — use adaptive thinking.**

\`thinking: {type: "enabled", budget_tokens: N}\` is deprecated on Opus 4.6 and Sonnet 4.6. Replace with \`thinking: {type: "adaptive"}\`, which lets Claude decide when and how much to think (and enables interleaved thinking automatically — no beta header).

\`\`\`python
# Old (still works on older models, deprecated on 4.6)
response = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    messages=[...]
)

# New (Opus 4.6 / Sonnet 4.6)
response = client.messages.create(
    model="claude-opus-4-6",  # or "claude-sonnet-4-6"
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # optional: low | medium | high | max
    messages=[...]
)
\`\`\`

Adaptive thinking is the long-term target and outperforms manual extended thinking on internal evaluations. Move when you can.

**Transitional escape hatch:** manual extended thinking is still *functional* on Opus 4.6 and Sonnet 4.6 (deprecated, removed in a future release). To bound token spend on a runaway workload before you've tuned \`effort\`, keep \`budget_tokens\` alongside an explicit \`effort\` value, then remove it in a follow-up. \`budget_tokens\` must be strictly less than \`max_tokens\`:

\`\`\`python
# Transitional only — deprecated, plan to remove
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16384,
    thinking={"type": "enabled", "budget_tokens": 8192},  # must be < max_tokens
    output_config={"effort": "medium"},
    messages=[...],
)
\`\`\`

If the user asks for a "thinking budget" on 4.6, prefer \`effort\` — \`low\`, \`medium\`, \`high\`, or \`max\` (Opus-tier only — not Sonnet or Haiku) rather than a token count.

**2. Effort parameter (Opus 4.5, Opus 4.6, Sonnet 4.6 only).**

Controls thinking depth and overall token spend. Goes inside \`output_config\`, not top-level. Default is \`high\`. \`max\` is Opus-tier only (Opus 4.6 and later). Errors on Sonnet 4.5 and Haiku 4.5.

\`\`\`python
output_config={"effort": "medium"}  # often the best cost / quality balance
\`\`\`

### Migrating to the 4.6 family (Opus 4.6 and Sonnet 4.6)

**3. Assistant-turn prefills return 400 (Opus 4.6 and Sonnet 4.6).**

Prefilled responses on the final assistant turn are unsupported on both Opus 4.6 and Sonnet 4.6 — both return 400. Adding assistant messages *elsewhere* (e.g. few-shot examples) still works. Pick the replacement matching what the prefill did:

| Prefill was used for                               | Replacement                                                                                                                               |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Forcing JSON / YAML / schema output                | \`output_config.format\` with a \`json_schema\` — see example below                                                                           |
| Forcing a classification label                     | Tool with an enum field containing valid labels, or structured outputs                                                                    |
| Skipping preambles (\`Here is the summary:\\n\`)      | System prompt instruction: *"Respond directly without preamble. Do not start with phrases like 'Here is...' or 'Based on...'."*           |
| Steering around bad refusals                       | Usually no longer needed — 4.6 refuses far more appropriately. Plain user-turn prompting is sufficient.                                   |
| Continuing an interrupted response                 | Move continuation into the user turn: *"Your previous response was interrupted and ended with \`[last text]\`. Continue from there."*     |
| Injecting reminders / context hydration            | Inject into the user turn instead. For complex agent harnesses, expose context via a tool call or during compaction.                      |

\`\`\`python
# Old (fails on Opus 4.6 / Sonnet 4.6) — prefill forcing JSON shape
messages=[
    {"role": "user", "content": "Extract the name."},
    {"role": "assistant", "content": "{\\"name\\": \\""},
]

# New — structured outputs replace the prefill
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    output_config={"format": {"type": "json_schema", "schema": {...}}},
    messages=[{"role": "user", "content": "Extract the name."}],
)
\`\`\`

**4. Stream for \`max_tokens > ~16K\` (all models); Opus 4.6 alone reaches 128K.**

Non-streaming requests hit SDK HTTP timeouts at high \`max_tokens\` regardless of model — stream for anything above ~16K output. Streamable ceiling differs: Sonnet 4.6 and Haiku 4.5 cap at 64K, Opus 4.6 goes up to 128K.

\`\`\`python
with client.messages.stream(model="claude-opus-4-6", max_tokens=64000, ...) as stream:
    message = stream.get_final_message()
\`\`\`

**5. Tool-call JSON escaping may differ (Opus 4.6 and Sonnet 4.6).**

Both 4.6 models can produce tool call \`input\` fields with Unicode or forward-slash escaping. Parse with \`json.loads()\` / \`JSON.parse()\` — don't raw-string-match the serialized input.

### All models

**6. \`output_format\` → \`output_config.format\` (API-wide).**

The top-level \`output_format\` parameter on \`messages.create()\` is deprecated. Use \`output_config.format\`. Applies to every model.

---

## Beta Headers to Remove on 4.6

Several beta headers required on 4.5 are now GA on 4.6 and should be removed. Leaving them is harmless but misleading; removing them lets you move from \`client.beta.messages.create(...)\` back to \`client.messages.create(...)\`.

| Header                                    | Status on 4.6                                              | Action                                                  |
| ----------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------- |
| \`effort-2025-11-24\`                       | Effort parameter is GA                                     | Remove                                                  |
| \`fine-grained-tool-streaming-2025-05-14\`  | GA                                                         | Remove                                                  |
| \`interleaved-thinking-2025-05-14\`         | Adaptive thinking enables interleaved thinking automatically | Remove when using adaptive thinking; still functional on Sonnet 4.6 *with* manual extended thinking, but that path is deprecated |
| \`token-efficient-tools-2025-02-19\`        | Built in to all Claude 4+ models                           | Remove (no effect)                                      |
| \`output-128k-2025-02-19\`                  | Built in to Claude 4+ models                               | Remove (no effect)                                      |

Once all are removed and you've moved to adaptive thinking, switch the call site from the beta namespace back to the regular one:

\`\`\`python
# Before
response = client.beta.messages.create(
    model="claude-opus-4-5",
    betas=["interleaved-thinking-2025-05-14", "effort-2025-11-24"],
    ...
)

# After
response = client.messages.create(
    model="claude-opus-4-6",
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    ...
)
\`\`\`

---

## Additional Changes When Coming from 3.x / 4.0 / 4.1 → 4.6

Jumping from Opus 4.1, Sonnet 4, Sonnet 3.7, or an older Claude 3.x model directly to 4.6: apply everything above *plus* this section. Users already on Opus 4.5 / Sonnet 4.5 can skip this.

**1. Sampling parameters: \`temperature\` OR \`top_p\`, not both.**

Passing both errors on every Claude 4+ model:

\`\`\`python
# Old (3.x only — errors on 4+)
client.messages.create(temperature=0.7, top_p=0.9, ...)

# New
client.messages.create(temperature=0.7, ...)  # or top_p, not both
\`\`\`

**2. Update tool versions.**

Legacy tool versions are unsupported on 4+. Both the \`type\` and the \`name\` field change — \`text_editor_20250728\` and \`str_replace_based_edit_tool\` are a pair; updating one without the other 400s. Also remove the \`undo_edit\` command:

| Old                                               | New                                                     |
| ------------------------------------------------- | ------------------------------------------------------- |
| \`text_editor_20250124\` + \`str_replace_editor\`     | \`text_editor_20250728\` + \`str_replace_based_edit_tool\`  |
| \`code_execution_*\` (earlier versions)             | \`code_execution_20250825\`                               |
| \`undo_edit\` command                               | *(no longer supported — delete call sites)*             |

\`\`\`python
# Before
tools = [{"type": "text_editor_20250124", "name": "str_replace_editor"}]

# After — BOTH fields change
tools = [{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}]
\`\`\`

**3. Handle the \`refusal\` stop reason.**

Claude 4+ can return \`stop_reason: "refusal"\`. If your code only handles \`end_turn\` / \`tool_use\` / \`max_tokens\`, add a branch:

\`\`\`python
if response.stop_reason == "refusal":
    # Surface the refusal to the user; do not retry with the same prompt
    ...
\`\`\`

**4. Handle the \`model_context_window_exceeded\` stop reason (4.5+).**

Distinct from \`max_tokens\`: it means the *context window* limit, not the requested output cap. Handle both:

\`\`\`python
if response.stop_reason == "model_context_window_exceeded":
    # Context window exhausted — compact or split the conversation
    ...
elif response.stop_reason == "max_tokens":
    # Requested output cap hit — retry with higher max_tokens or stream
    ...
\`\`\`

**5. Trailing newlines preserved in tool call string parameters (4.5+).**

4.5 and 4.6 preserve trailing newlines older models stripped. If tool implementations exact-match against tool-call \`input\` values (e.g. \`if name == "foo"\`), verify they still match when the model sends \`"foo\\n"\`. Normalizing with \`.rstrip()\` on the receiving side is usually the simplest fix.

**6. Haiku: rate limits reset between generations.**

Haiku 4.5 has its own rate-limit pool separate from Haiku 3 / 3.5. If ramping traffic, check your tier's Haiku 4.5 limits at [API rate limits](https://platform.claude.com/docs/en/api/rate-limits) — a quota that served Haiku 3.5 may need a tier bump for the same volume on 4.5.

---

## Prompt-Behavior Changes (Opus 4.5 / 4.6, Sonnet 4.6)

These don't break code, but prompts that worked on 4.5-and-earlier may over- or under-trigger on 4.6. Tune as needed.

**1. Aggressive instructions cause overtriggering.** Opus 4.5 and 4.6 follow the system prompt much more closely. Prompts written to *overcome* the old reluctance are now too aggressive:

| Before (worked on 4.0 / 4.5)                | After (use on 4.6)                        |
| ------------------------------------------- | ----------------------------------------- |
| \`CRITICAL: You MUST use this tool when...\`  | \`Use this tool when...\`                   |
| \`Default to using [tool]\`                   | \`Use [tool] when it would improve X\`      |
| \`If in doubt, use [tool]\`                   | *(delete — no longer needed)*             |

If the model now overtriggers a tool or skill, the fix is almost always to dial back the language, not add more guardrails.

**2. Overthinking and excessive exploration (Opus 4.6).** At higher \`effort\`, Opus 4.6 explores more before answering. If that burns too many thinking tokens, lower \`effort\` first (\`medium\` is often the sweet spot) before adding prose to constrain reasoning.

**3. Overeager subagent spawning (Opus 4.6).** Opus 4.6 strongly prefers delegating to subagents. If it spawns one for something a direct \`grep\` or \`read\` would solve, add: *"Use subagents only for parallel or independent workstreams. For single-file reads or sequential operations, work directly."*

**4. Overengineering (Opus 4.5 / 4.6).** Both may add extra files, abstractions, or defensive error handling beyond what was asked. For minimal changes: *"Only make changes directly requested. Don't add helpers, abstractions, or error handling for scenarios that can't happen."*

**5. LaTeX math output (Opus 4.6).** Opus 4.6 defaults to LaTeX (\`\\frac{}{}\`, \`$...$\`) for math and technical content. For plain text: *"Format all math as plain text — no LaTeX, no \`$\`, no \`\\frac{}{}\`. Use \`/\` for division and \`^\` for exponents."*

**6. Skipped verbal summaries (4.6 family).** The 4.6 models are more concise and may skip the summary paragraph after a tool call. If you rely on those summaries: *"After completing a task that involves tool use, provide a brief summary of what you did."*

**7. "Think" as a trigger word (Opus 4.5 with thinking disabled).** When \`thinking\` is off, Opus 4.5 is sensitive to the word *think* and may reason more than wanted. Use \`consider\`, \`evaluate\`, or \`reason through\` instead.

---

## Model-ID Rename Quick Reference

| Old string (migration source)  | New string         |
| ------------------------------ | ------------------ |
| \`claude-opus-4-7\`              | \`claude-opus-4-8\`  |
| \`claude-opus-4-6\`              | \`claude-opus-4-8\`  |
| \`claude-opus-4-5\`              | \`claude-opus-4-8\`  |
| \`claude-opus-4-1\`              | \`claude-opus-4-8\`  |
| \`claude-opus-4-0\`              | \`claude-opus-4-8\`  |
| \`claude-sonnet-4-5\`            | \`claude-sonnet-4-6\`|
| \`claude-sonnet-4-0\`            | \`claude-sonnet-4-6\`|

Older aliases (\`claude-opus-4-7\`, \`claude-opus-4-6\`, \`claude-opus-4-5\`, \`claude-sonnet-4-5\`, etc.) are still active and can be pinned if you need time before upgrading — see \`shared/models.md\` for the full legacy list.

### Amazon Bedrock model IDs

If the code uses the \`AnthropicBedrockMantle\` client (Python \`anthropic[bedrock]\`, TypeScript \`@anthropic-ai/bedrock-sdk\`, Java \`BedrockMantleBackend\`, Go \`bedrock.NewMantleClient\`, etc.) or targets \`https://bedrock-mantle.{region}.api.aws/anthropic\`, it runs on **Claude in Amazon Bedrock**. All breaking changes in this guide apply unchanged — same Messages API shape — but model IDs carry an \`anthropic.\` provider prefix:

| First-party ID | Bedrock ID |
|---|---|
| \`claude-opus-4-8\` | \`anthropic.claude-opus-4-8\` |
| \`claude-opus-4-7\` | \`anthropic.claude-opus-4-7\` |
| \`claude-haiku-4-5\` | \`anthropic.claude-haiku-4-5\` |

When migrating a Bedrock file, apply the same rename-table row as first-party, then keep/add the \`anthropic.\` prefix. Don't generate a first-party \`claude-*\` ID for a Bedrock client — it will 400.

**Skip for Bedrock:** the \`code_execution_*\` tool-version item and the **Task Budgets** section — both are first-party-only (Bedrock doesn't support server-side Anthropic tools or the \`task-budgets-2026-03-13\` beta). Everything else — \`effort\`, adaptive/extended thinking, \`output_config.format\`, \`thinking.display\`, fine-grained tool streaming, token counting — is available on Bedrock.

> **Out of scope:** the legacy Amazon Bedrock integration (\`InvokeModel\` / \`Converse\` APIs with ARN-versioned IDs like \`anthropic.claude-3-5-sonnet-20241022-v2:0\`) uses a different request shape and model-ID format. This guide doesn't cover it; WebFetch the Bedrock page in \`shared/live-sources.md\` if migrating between the two Bedrock integrations.

### Claude Platform on AWS

If the code uses \`AnthropicAWS\` / \`AnthropicAws\` / \`anthropicaws.NewClient\` / \`AnthropicAwsClient\` (or targets \`https://aws-external-anthropic.{region}.api.aws\`), it runs on **Claude Platform on AWS** — Anthropic-operated, same-day API parity. Model IDs are bare first-party strings; apply the rename table above verbatim and every breaking-change section unchanged. Nothing to skip. Don't add an \`anthropic.\` prefix (that's Amazon Bedrock, a separate offering). See \`shared/claude-platform-on-aws.md\` for client/auth details.

---

## Migration Checklist

\`[BLOCKS]\` items cause a 400 error, infinite loop, silent timeout, or wrong tool selection if missed — apply these as code edits, not suggestions. \`[TUNE]\` items are quality/cost adjustments.

For each file that calls \`messages.create()\` / equivalent SDK method:

- [ ] **[BLOCKS]** Update the \`model=\` string to the new alias
- [ ] **[BLOCKS]** Replace \`budget_tokens\` with \`thinking={"type": "adaptive"}\` (deprecated on Opus 4.6 / Sonnet 4.6)
- [ ] **[BLOCKS]** Move \`format\` from top-level \`output_format\` into \`output_config.format\`
- [ ] **[BLOCKS]** Remove any assistant-turn prefills if targeting Opus 4.6 or Sonnet 4.6 (see the prefill replacement table)
- [ ] **[BLOCKS]** Switch to streaming if \`max_tokens > ~16000\` (otherwise SDK HTTP timeout)
- [ ] **[TUNE]** Verify tool-input handling parses JSON rather than raw-string-matching the serialized input (4.6 may escape Unicode / forward slashes differently)
- [ ] **[TUNE]** Set \`output_config={"effort": "..."}\` explicitly — especially moving Sonnet 4.5 → Sonnet 4.6 (4.6 defaults to \`high\`)
- [ ] **[TUNE]** Remove GA beta headers: \`effort-2025-11-24\`, \`fine-grained-tool-streaming-2025-05-14\`, \`token-efficient-tools-2025-02-19\`, \`output-128k-2025-02-19\`; remove \`interleaved-thinking-2025-05-14\` once on adaptive thinking
- [ ] **[TUNE]** Switch \`client.beta.messages.create(...)\` → \`client.messages.create(...)\` once all betas are removed
- [ ] **[TUNE]** Review system prompt for aggressive tool language (\`CRITICAL:\`, \`MUST\`, \`If in doubt\`) and dial it back

**Extra items when coming from 3.x / 4.0 / 4.1:**
- [ ] **[BLOCKS]** Remove either \`temperature\` or \`top_p\` (passing both 400s on Claude 4+)
- [ ] **[BLOCKS]** Update text-editor tool \`type\` to \`text_editor_20250728\`
- [ ] **[BLOCKS]** Update text-editor tool \`name\` to \`str_replace_based_edit_tool\` — changing only the \`type\` and keeping \`name: "str_replace_editor"\` returns a 400
- [ ] **[BLOCKS]** Update code-execution tool to \`code_execution_20250825\`
- [ ] **[BLOCKS]** Delete any \`undo_edit\` command call sites
- [ ] **[TUNE]** Add handling for \`stop_reason == "refusal"\`
- [ ] **[TUNE]** Add handling for \`stop_reason == "model_context_window_exceeded"\` (4.5+)
- [ ] **[TUNE]** Verify tool-param string matching tolerates trailing newlines (preserved on 4.5+)
- [ ] **[TUNE]** If moving to Haiku 4.5: review rate-limit tier (separate pool from Haiku 3.x)

**Verification:**
- [ ] Run one test request and inspect \`response.stop_reason\`, \`response.usage\`, and whether tool-use / thinking behavior matches expectations

For cached prompts: render order and hash inputs didn't change, so existing \`cache_control\` breakpoints keep working. Changing the model string invalidates the existing cache — the first request on the new model writes the cache fresh.

---

## Migrating to Opus 4.7

> **Model ID \`claude-opus-4-7\` is authoritative as written here.** When the user asks to migrate to Opus 4.7, write \`model="claude-opus-4-7"\` exactly. Do not WebFetch to verify — this guide is the source of truth for migration target IDs. The corresponding entry exists in \`shared/models.md\`.

Claude Opus 4.7 was Anthropic's most capable model at its launch and is now the previous-generation Opus (Opus 4.8 is current — see Migrating to Opus 4.8). Highly autonomous; strong on long-horizon agentic work, knowledge work, vision, and memory. This section is the layered breaking-change path for callers coming from Opus 4.6 or older — layered on top of the 4.6 migration above. From Opus 4.5 or older: apply 4.6 first, then this section, then the 4.8 section.

**TL;DR for someone already on Opus 4.6:** update the model ID to \`claude-opus-4-7\`, strip any remaining \`budget_tokens\` and sampling parameters (both 400 on Opus 4.7), give \`max_tokens\` extra headroom and re-baseline with \`count_tokens()\`, opt back into \`thinking.display: "summarized"\` if reasoning is surfaced to users, and re-tune \`effort\`.

### Breaking changes (will 400 on Opus 4.7)

**Extended thinking removed.**

\`thinking: {type: "enabled", budget_tokens: N}\` is unsupported on Claude Opus 4.7 or later and returns 400. Switch to adaptive thinking (\`thinking: {type: "adaptive"}\`) and use the effort parameter to control depth. Adaptive thinking is off by default on Opus 4.7: requests with no \`thinking\` field run without thinking. Set \`thinking: {type: "adaptive"}\` explicitly to enable it.

\`\`\`python
# Before (Opus 4.6)
client.messages.create(
    model="claude-opus-4-6",
    max_tokens=64000,
    thinking={"type": "enabled", "budget_tokens": 32000},
    messages=[{"role": "user", "content": "..."}],
)

# After (Opus 4.7)
client.messages.create(
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # or "max", "xhigh", "medium", "low"
    messages=[{"role": "user", "content": "..."}],
)
\`\`\`

If the caller wasn't using extended thinking, no change is required — thinking is off by default, or set explicitly with \`thinking={"type": "disabled"}\`.

Delete \`budget_tokens\` plumbing entirely. For the replacement \`effort\` value, see **Choosing an effort level on Opus 4.7** — there's no exact 1:1 mapping from \`budget_tokens\`.

**Sampling parameters removed.**

\`temperature\`, \`top_p\`, and \`top_k\` are no longer accepted on Claude Opus 4.7; requests with them return 400. Remove them. Steer behavior with prompting. If you used \`temperature = 0\` for determinism, note it never guaranteed identical outputs on prior models.

\`\`\`python
# Before — errors on Opus 4.7
client.messages.create(temperature=0.7, top_p=0.9, ...)

# After
client.messages.create(...)  # no sampling params
\`\`\`

- **If the intent was determinism** — use \`effort: "low"\` with a tighter prompt.
- **If the intent was creative variance** — the replacement depends on the use case; ask the user how they want variance elicited. If you can't ask, add a use-case-appropriate instruction like *"choose something off-distribution and interesting"* — e.g. for text, *"Vary your phrasing and structure across responses"*; for frontend/design, use the propose-4-directions approach under **Design and frontend coding**.

### Choosing an effort level on Opus 4.7

\`budget_tokens\` controlled how much to *think*; \`effort\` controls how much to think *and* act, so there's no exact 1:1 mapping. Use \`xhigh\` for best results in coding and agentic use cases, and a minimum of \`high\` for most intelligence-sensitive use cases. Experiment with other levels to tune token usage and intelligence:

| Level | Use when | Notes |
| --- | --- | --- |
| \`max\` | Intelligence-demanding tasks worth testing at the ceiling | Can deliver gains but may show diminishing returns from increased token usage; can be prone to overthinking |
| \`xhigh\` | **Most coding and agentic use cases** | The best setting for these; the default in Claude Code |
| \`high\` | Intelligence-sensitive use cases generally | Balances token usage and intelligence; recommended minimum for intelligence-sensitive work |
| \`medium\` | Cost-sensitive use cases that reduce token usage while trading off intelligence | |
| \`low\` | Short, scoped tasks and latency-sensitive workloads that aren't intelligence-sensitive | |

### Silent default changes (no error, but behavior differs)

**Thinking content omitted by default.**

Thinking blocks still appear in the response stream on Opus 4.7, but the \`thinking\` field is empty unless you opt in. This is a silent change from Opus 4.6 (which defaulted to summarized thinking text). To restore summarized thinking, set \`thinking.display\` to \`"summarized"\`. The block-field name is unchanged — still \`block.thinking\` on a \`thinking\`-type block.

**Detect this:** any code reading \`block.thinking\` from a \`thinking\`-type block and rendering it in a UI, log, or trace. The fix is the request parameter, not the response handling — add \`display: "summarized"\`:

\`\`\`python
thinking={"type": "adaptive", "display": "summarized"}  # values: "omitted" (default) | "summarized"
\`\`\`

The default is \`"omitted"\` on Opus 4.7. If thinking content was never surfaced, no change needed. If your product streams reasoning to users, the new default appears as a long pause before output; set \`display: "summarized"\` to restore visible progress.

**Updated token counting.**

Opus 4.7 and Opus 4.6 count tokens differently — the same input text produces a higher token count on 4.7, and \`/v1/messages/count_tokens\` returns different numbers. Token efficiency varies by workload shape. Prompting interventions, \`task_budget\`, and \`effort\` can control costs (they may trade off intelligence). Update your \`max_tokens\` parameters to give additional headroom, including compaction triggers. Opus 4.7 provides a 1M context window at standard API pricing with no long-context premium.

What else to check:

- Client-side token estimators (tiktoken-style) calibrated against 4.6
- Cost calculators multiplying tokens by a fixed per-token rate
- Rate-limit retry thresholds keyed to measured token counts

Re-baseline by re-running \`client.messages.count_tokens()\` against \`claude-opus-4-7\` on a representative sample. Don't apply a blanket multiplier. For cost-sensitive workloads, consider reducing \`effort\` by one level. For agentic loops, consider Task Budgets (below).

### New feature: Task Budgets (beta)

Opus 4.7 introduces **task budgets** — tell Claude how many tokens it has for a full agentic loop (thinking + tool calls + final output). The model sees a running countdown and uses it to prioritize and wrap up gracefully.

This is a suggestion the model is aware of, not a hard cap. Distinct from \`max_tokens\`, which remains the enforced per-response limit and is *not* surfaced to the model. Use \`task_budget\` to have the model self-moderate; use \`max_tokens\` as a hard ceiling.

Requires beta header \`task-budgets-2026-03-13\`:

\`\`\`python
client.beta.messages.create(
    betas=["task-budgets-2026-03-13"],
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={
        "effort": "high",
        "task_budget": {"type": "tokens", "total": 128000},
    },
    messages=[...],
)
\`\`\`

Set a generous budget for open-ended agentic tasks, tighter for latency-sensitive ones. Minimum \`task_budget.total\` is 20,000 tokens. If too restrictive, the model may complete less thoroughly, referencing its budget. Don't add \`task_budget\` during a migration unless you're sure the value is right — measure if you can run the workload; otherwise ask. This is the primary lever for offsetting the token-counting shift on agentic workloads.

### Capability improvements

**High-resolution vision.** Opus 4.7 is the first Claude with high-resolution image support — max **2576 pixels on the long edge** (up from 1568px on Opus 4.6 and prior). Gains on vision-heavy workloads, especially computer use and screenshot/artifact/document understanding. Coordinates returned now map 1:1 to actual image pixels — no scale-factor math. High-res is automatic on Opus 4.7 — no beta header, no client-side opt-in.

**Token cost.** Full-resolution images on Opus 4.7 can use up to ~3× more image tokens than prior models (up to ~4784 tokens/image vs the previous ~1,600-token cap). If the extra fidelity isn't needed, downsample client-side before sending — but don't add downsampling by default during a migration. If unsure whether the pipeline needs the fidelity, ask. Use \`count_tokens()\` on representative images to re-baseline before reacting to a cost shift.

Beyond resolution, Opus 4.7 also improves low-level perception (pointing, measuring, counting) and natural-image bounding-box localization.

**Knowledge work.** Gains where the model visually verifies its own output — \`.docx\` redlining, \`.pptx\` editing, programmatic chart/figure analysis. If prompts have scaffolding like *"double-check the slide layout before returning"*, try removing it and re-baselining.

**Memory.** Opus 4.7 is better at writing and using file-system-based memory. Agents maintaining a scratchpad, notes file, or structured memory store should improve.

**User-facing progress updates.** Opus 4.7 provides more regular, higher-quality interim updates during long agentic traces. If the prompt has scaffolding like *"After every 3 tool calls, summarize progress"*, try removing it. If updates aren't well-calibrated to your use case, describe what they should look like in the prompt with examples.

### Real-time cybersecurity safeguards

Requests involving prohibited or high-risk topics may lead to refusals.

### Fast Mode: not available on Opus 4.7

Opus 4.7 has no Fast Mode variant. Opus 4.6 Fast remains supported. Only surface this if the caller's code uses a Fast Mode model string (e.g. \`claude-opus-4-6-fast\`); if "fast" doesn't appear in the code, say nothing about Fast Mode.

When you see \`model="claude-opus-4-6-fast"\` (or similar), the migration edit is:

\`\`\`python
# Opus 4.7 has no Fast Mode — keeping on 4.6 Fast (caller's choice to switch to standard Opus 4.7).
model="claude-opus-4-6-fast",
\`\`\`

Leave the model string unchanged, add the comment, and tell the user their two options — (a) stay on Opus 4.6 Fast, which remains supported, or (b) move latency-tolerant traffic to standard Opus 4.7 for the intelligence gain. Don't rewrite the model string to \`claude-opus-4-7\` yourself; that silently trades latency for intelligence, which is the caller's decision.

### Behavioral shifts (prompt-tunable)

These don't break anything, but prompts tuned for Opus 4.6 may land differently. Opus 4.7 is more steerable than 4.6, so small nudges usually close the gap.

**More literal instruction following.** Opus 4.7 interprets prompts more literally and explicitly than 4.6, especially at lower effort. It won't silently generalize an instruction from one item to another, and won't infer requests you didn't make. The upside is precision and less thrash — better for API use cases with carefully tuned prompts, structured extraction, and predictable pipelines. A prompt and harness review is especially helpful for this migration.

**Verbosity calibrates to task complexity.** Opus 4.7 scales response length to judged task complexity — shorter on simple lookups, longer on open-ended analysis. If the product depends on a particular length, tune the prompt explicitly. To reduce verbosity:

> *"Provide concise, focused responses. Skip non-essential context, and keep examples minimal."*

For specific over-verbosity (e.g. over-explaining), add targeted instructions. Positive examples of desired concision tend to work better than negative instructions. Don't assume existing "be concise" instructions should be removed — test first.

**Tone and writing style.** Opus 4.7 is more direct and opinionated, with less validation-forward phrasing and fewer emoji than 4.6's warmer style. Prose style on long-form writing may shift. If the product relies on a specific voice, re-evaluate style prompts. For a warmer voice:

> *"Use a warm, collaborative tone. Acknowledge the user's framing before answering."*

**\`effort\` matters more than on any prior Opus.** Opus 4.7 respects \`effort\` levels more strictly, especially at the low end. At \`low\`/\`medium\` it scopes work to what was asked — good for latency and cost, but on moderate tasks at \`low\` there's some risk of under-thinking.

- If shallow reasoning shows up on complex problems, raise \`effort\` to \`high\`/\`xhigh\` rather than prompting around it.
- If \`effort\` must stay \`low\` for latency: *"This task involves multi-step reasoning. Think carefully through the problem before responding."*
- At \`xhigh\`/\`max\`, set a large \`max_tokens\` so the model has room to think and act across tool calls and subagents. Start at 64K and tune. (\`xhigh\` is a new effort level on Opus 4.7, between \`high\` and \`max\`.)

Adaptive-thinking triggering is steerable. If it thinks more often than wanted (can happen with large/complex system prompts): *"Thinking adds latency and should only be used when it will meaningfully improve answer quality — typically for problems that require multi-step reasoning. When in doubt, respond directly."*

**Uses tools less often by default.** Opus 4.7 uses tools less and reasoning more. Better in most cases, but for products relying on tools (search/retrieval, function-calling, computer-use) it can drop tool-use rate. Two levers:

- **Raise \`effort\`** — \`high\`/\`xhigh\` show substantially more tool usage in agentic search and coding.
- **Prompt for it** — be explicit in tool descriptions or the system prompt about when and how to use the tool:

> *"When the answer depends on information not present in the conversation, call the \`search\` tool before answering — do not answer from prior knowledge."*

**Fewer subagents by default.** Opus 4.7 spawns fewer subagents than 4.6. Steerable — give explicit guidance on when delegation is desirable. For a coding agent:

> *"Do not spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see). Spawn multiple subagents in the same turn when fanning out across items or reading multiple files."*

**Design and frontend coding.** Opus 4.7 has stronger design instincts than 4.6, with a consistent default house style: warm cream/off-white backgrounds (around \`#F4F1EA\`), serif display type (Georgia, Fraunces, Playfair), italic word-accents, and a terracotta/amber accent. Reads well for editorial/hospitality/portfolio briefs, but feels off for dashboards, dev tools, fintech, healthcare, or enterprise apps — and appears in slide decks as well as web UIs.

The default is persistent. Generic instructions ("don't use cream," "make it clean and minimal") tend to shift to a different fixed palette rather than producing variety. Two approaches work reliably:

1. **Specify a concrete alternative.** The model follows explicit specs precisely — give exact hex values, typefaces, and layout constraints.
2. **Have the model propose options before building:**

   > *"Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface — one-line rationale). Ask the user to pick one, then implement only that direction."*

If the caller previously relied on \`temperature\` for design variety, use approach (2) — it produces meaningfully different directions across runs.

Opus 4.7 also needs less anti-slop prompting than previous models. A short nudge works alongside the variety approaches above:

> *"Never use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions."*

**Interactive coding products.** Opus 4.7's token usage differs between autonomous single-turn coding agents and interactive multi-turn ones — it uses more tokens in interactive settings because it reasons more after user turns. This can improve long-horizon coherence and instruction following in long interactive sessions, with more token usage. To maximize both performance and efficiency, use \`effort: "xhigh"\` or \`"high"\`, add autonomous features (like an auto mode), and reduce required human interactions.

When limiting required interactions, specify the task, intent, and relevant constraints upfront in the first human turn. Well-specified descriptions upfront maximize autonomy and intelligence while minimizing extra token usage after user turns. Ambiguous prompts conveyed progressively over multiple turns reduce efficiency and sometimes performance.

**Code review.** Opus 4.7 is meaningfully better at finding bugs, with higher recall and precision. But a harness tuned for an earlier model may initially show *lower* recall — a harness effect, not a regression. When a review prompt says "only report high-severity issues," "be conservative," or "don't nitpick," Opus 4.7 follows that more faithfully: it investigates just as thoroughly, finds the bugs, then declines to report findings below the stated bar. Precision rises, measured recall can fall even though bug-finding improved.

Recommended prompt language:

> *"Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage — a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them."*

This works without an actual second step, but moving confidence filtering out of the finding step often helps. If the harness has a separate verification/dedup/ranking stage, tell the model its job at the finding stage is coverage, not filtering. For single-pass self-filtering, be concrete about the bar rather than qualitative terms like "important" — e.g. *"report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences."* Iterate on prompts against a subset of evals.

**Computer use.** Works across resolutions up to the new 2576px / 3.75MP maximum. Sending images at **1080p** balances performance and cost; **720p** or **1366×768** are lower-cost options with strong performance. Test for the ideal settings; experimenting with \`effort\` can help.

---

## Opus 4.7 Migration Checklist

\`[BLOCKS]\` items cause a 400, infinite loop, silent truncation, or empty output if missed — apply as code edits. \`[TUNE]\` items are quality/cost adjustments — surface as recommendations.

\`[BLOCKS]\` items prefixed with "If…" or "At…" are conditional. Before working through the list, scan the file: does it surface thinking text to a UI/log? Does it set \`output_config.effort\` to \`"xhigh"\` or \`"max"\`? Is it a multi-turn agentic loop? Apply only the items whose condition matches.

- [ ] **[BLOCKS]** Replace \`thinking: {type: "enabled", budget_tokens: N}\` with \`thinking: {type: "adaptive"}\` + \`output_config.effort\`; delete \`budget_tokens\` plumbing entirely
- [ ] **[BLOCKS]** Strip \`temperature\`, \`top_p\`, \`top_k\` from request construction
- [ ] **[BLOCKS]** If thinking content is surfaced to users or stored in logs: add \`thinking.display: "summarized"\` (otherwise the rendered text is empty)
- [ ] **[BLOCKS]** At \`output_config.effort\` of \`xhigh\` or \`max\`: set \`max_tokens\` ≥ 64000 (otherwise output truncates mid-thought)
- [ ] **[TUNE]** Give \`max_tokens\` and compaction triggers extra headroom; re-run \`count_tokens()\` against \`claude-opus-4-7\` on representative prompts (no blanket multiplier)
- [ ] **[TUNE]** Re-baseline cost and rate-limit dashboards *before* reacting to measured shifts
- [ ] **[TUNE]** Re-evaluate \`effort\` per route — \`xhigh\` for coding/agentic, minimum \`high\` for most intelligence-sensitive work
- [ ] **[TUNE]** Multi-turn agentic loops: adopt Task Budgets (\`output_config.task_budget\`, beta \`task-budgets-2026-03-13\`, minimum 20k tokens) for capping *cumulative* spend across a loop; per-turn depth is \`effort\`
- [ ] **[TUNE]** Check for ambiguous instructions that relied on 4.6 generalizing intent; make them clearer — 4.7 follows them literally
- [ ] **[TUNE]** Tool-use workloads: add explicit when/how-to-use guidance to tool descriptions (4.7 reaches for tools less often)
- [ ] **[TUNE]** Verbosity: test existing length instructions before changing them — 4.7 calibrates length to task complexity
- [ ] **[TUNE]** Remove forced-progress-update scaffolding (*"after every N tool calls…"*)
- [ ] **[TUNE]** Remove knowledge-work verification scaffolding (*"double-check the slide layout…"*) and re-baseline
- [ ] **[TUNE]** Add a tone instruction if a warmer voice is needed; re-evaluate style prompts on writing-heavy routes
- [ ] **[TUNE]** Subagent tool present: add explicit spawn / don't-spawn guidance
- [ ] **[TUNE]** Frontend/design output: specify a concrete palette/typeface, or have the model propose 4 visual directions before building (the default cream/serif house style is persistent)
- [ ] **[TUNE]** Interactive coding products: use \`effort: "xhigh"\` or \`"high"\`, add autonomous features (e.g. an auto mode), and specify task/intent/constraints upfront in the first turn
- [ ] **[TUNE]** Code-review harnesses: remove or loosen "only report high-severity" / "be conservative" filters; have the model report every finding with confidence + severity; move filtering downstream
- [ ] **[TUNE]** Vision-heavy pipelines: leave images at native resolution up to 2576px long edge; remove scale-factor math from coordinate handling (coords are 1:1 with pixels). No beta header / opt-in needed — high-res is automatic on Opus 4.7.
- [ ] **[TUNE]** Computer-use pipelines: send screenshots at 1080p for a good performance/cost balance (720p or 1366×768 for cost-sensitive); experiment with \`effort\`
- [ ] **[TUNE]** Cost-sensitive image pipelines: full-res images on 4.7 use up to ~4784 tokens vs ~1,600 on prior models. Downsampling client-side avoids the increase, but don't downsample by default — if unsure whether fidelity is needed, ask. Re-baseline with \`count_tokens()\` first.

---

## Migrating to Opus 4.8

> **Model ID \`claude-opus-4-8\` is authoritative as written here.** When the user asks to migrate to Opus 4.8, write \`model="claude-opus-4-8"\` exactly. Do not WebFetch to verify — this guide is the source of truth for migration target IDs. The corresponding entry exists in \`shared/models.md\`.

Claude Opus 4.8 is the most capable generally available model to date — highly autonomous, with state-of-the-art long-horizon agentic execution, knowledge work, and memory. Layered on top of the Opus 4.7 migration above. From Opus 4.6 or older: apply the 4.6 and 4.7 sections first, then this one.

**No new breaking changes.** Opus 4.8 keeps the same request surface as Opus 4.7 — adaptive thinking only (\`budget_tokens\` still 400s; use \`{type: "adaptive"}\`), sampling parameters still rejected, last-assistant-turn prefills still 400, \`thinking.display\` still defaults to \`"omitted"\`, and the \`low\`/\`medium\`/\`high\`/\`xhigh\`/\`max\` effort levels, Task Budgets (beta), and high-resolution vision all behave as on 4.7. A 4.7 → 4.8 migration is the model-ID swap plus prompt re-tuning — no required code edit beyond the model string.

**TL;DR for someone already on Opus 4.7:** swap the model ID to \`claude-opus-4-8\`. Nothing else is required to avoid an error. Then re-tune prompts for the behavioral shifts: 4.8 narrates *more* than 4.7 (add a silence-default for 4.7-like terseness), writes in a warmer, less hedged voice, is more deliberate and asks more often (add autonomy guidance to claw back ask-rate), and is more conservative about reaching for search, subagents, file-based memory, and custom tools (add explicit "when to use this" triggering). For long-horizon agentic work, give the full task spec up front in one well-specified turn and run at high effort.

### No new API breaking changes (inherited from 4.7)

These carry over from Opus 4.7 unchanged — apply only if coming from Opus 4.6 or earlier (see **Migrating to Opus 4.7** for before/after and SDK-specific syntax):

- \`thinking: {type: "enabled", budget_tokens: N}\` → 400. Use \`thinking: {type: "adaptive"}\` + \`output_config.effort\`.
- \`temperature\`, \`top_p\`, \`top_k\` → 400. Remove them; steer with prompting.
- Last-assistant-turn prefills → 400. Use \`output_config.format\` (structured outputs) or a system-prompt instruction.
- \`thinking.display\` defaults to \`"omitted"\`; set \`"summarized"\` if you surface reasoning to users.

If already on Opus 4.7 and these are clean, there's nothing to change here.

### New API feature: mid-session system prompts

You can deliver trusted instructions partway through a session by placing \`{"role": "system", ...}\` entries directly in the \`messages\` array — without editing the top-level system prompt and invalidating your prompt cache. Use it for things the application learns mid-session: async context arrived, a mode toggled (auto-approve enabled), files changed on disk, the remaining token budget dropped.

\`\`\`python
messages=[
    {"role": "user", "content": [{"type": "tool_result", "tool_use_id": "...", "content": "..."}]},
    {"role": "system", "content": "This project's codebase is Go. Write code in Go."},
]
\`\`\`

Phrase these as **context, not commands**. State the fact and let Claude act on it; avoid override-style language ("ignore what the user said", "regardless of the user's request", "disregard the previous instruction"). Claude is trained to protect users from instructions that appear to work against them, and that protection applies to the system role too. This is a beta (\`anthropic-beta: mid-conversation-system-2026-04-07\`), available from Opus 4.7 onward, not 4.8-exclusive. For cache-placement details and the older-model \`<system-reminder>\` fallback, see \`shared/prompt-caching.md\` and \`shared/agent-design.md\`.

### Capability improvements

**Long-horizon agentic execution.** Opus 4.8 is state-of-the-art at long, autonomous agentic work — complex refactors and overnight coding runs that complete without human correction. To get the most out of it, give the full task spec up front in a single well-specified initial turn and run at high effort (\`effort: "high"\` or \`"xhigh"\`). Its long-horizon coherence comes partly from reasoning more at each step; combined with a clear up-front goal, that planning often produces more efficient *and* more accurate output than prior frontier models. The "clear goal up front" principle maps to two product surfaces: in Claude Code, \`/goal\` sets direction for the run; with **Managed Agents (CMA)**, state what "done" looks like via an **Outcome** (\`user.define_outcome\` with a gradeable rubric — the harness runs an iterate → grade → revise loop), see \`shared/managed-agents-outcomes.md\`.

**Effort is a dimension to test, not a fixed setting.** On prior models many reached for \`xhigh\` reflexively to maximize intelligence. Opus 4.8 has a higher intelligence ceiling, so start at \`high\` as the default and iterate rather than defaulting to \`xhigh\`. Sweep \`medium\`, \`high\`, and \`xhigh\` on your own eval set and weigh the intelligence ↔ latency ↔ cost tradeoff per route — the relationship isn't monotonic: higher effort up front often *reduces* turn count and total cost on agentic work, while for some tasks \`medium\` delivers equally good results faster. Reserve \`max\` for extremely hard, latency-insensitive cases. The per-level effort table in **Migrating to Opus 4.7** applies unchanged on 4.8.

**Writing voice and clarity.** Testers consistently describe 4.8's prose as clearer, warmer, and less hedged than prior models, with fewer measurable AI vocal tics — especially at higher effort, where it approaches expert-level prose and structure. This is roughly the opposite direction from the 4.7 shift (4.7 was more clipped, direct, less validation-forward). If you added style prompts to counter 4.7's terseness or inject warmth, re-evaluate them against the new baseline before keeping them — they may now overcorrect. 4.8 is also a stronger thought partner: more thoughtful, more willing to push back, more likely to infer the right answer from context.

**Code review and debugging.** Stronger real-bug finding and clearer explanations than 4.7 — one-shot fixes where 4.7 needed more, and correctly identifying intermittent flakes rather than declaring "fixed" after one clean run. The 4.7 caveat still applies: if a review harness says "only report high-severity issues" or "be conservative", 4.8 follows it literally and measured recall can drop even though bug-finding improved. Tell the model to report everything and filter downstream (or review a second time) — see the **Code review** guidance in the 4.7 section.

### Behavioral shifts (prompt-tunable)

None break code, but prompts tuned for Opus 4.7 may land differently. 4.8 follows instructions well, so small explicit nudges close the gap.

**Tool triggering is surface-dependent (search & knowledge).** With a system prompt present, 4.8 is high-precision / low-recall — web search triggers slightly more often but runs fewer rounds per trigger, while knowledge-retrieval tools (Drive, project knowledge, connected files) trigger *less* often. It searches when confident search is needed and otherwise answers from context, which can lower research depth on tasks that need it. Recover should-search rate with an explicit search-first instruction:

> \`\`\`
> <search_first>
> For questions where current information would change the answer (recent events, current roles or prices, version-specific behavior, or anything the user flags as time-sensitive) search before answering rather than answering from memory. For open-ended research requests, begin searching immediately; do not ask a scoping question first unless the request is genuinely ambiguous about what to research.
> </search_first>
> \`\`\`

**Under-utilization of subagents, memory, and custom tools.** Separately from search, 4.8 is conservative about capabilities that need an explicit "decide to use this" step — file-based memory, subagent delegation, custom tools. It won't reach for complex or expensive capabilities unless reasonably sure they're needed. Steerable since 4.8 follows instructions well — say *when* each capability applies, not just that it exists:

> *"Before any task longer than a few turns, check your memory file for relevant prior context and write new findings to it as you go. When a task fans out across independent items (many files to read, many tests to run, many candidates to check), delegate to subagents rather than iterating serially."*

The same lever works at the tool-description level, not just the system prompt: a prescriptive description that states *when* to call a tool (e.g. "Call this when the user asks about current prices or recent events") gives meaningful lift on 4.8 over one that only states what the tool does. Make the trigger condition part of each tool's own \`description\`.

**More user-facing narration.** 4.8 narrates more than 4.7 — more text between tool calls in long tool-calling sessions, and longer, more detailed end-of-task wrap-ups by default. If you previously added scaffolding to force interim status ("after every 3 tool calls, summarize progress"), remove it — 4.8 does this on its own. If the narration is too verbose for a coding agent, an explicit silence-default makes it behave like 4.7 with no loss of quality:

> *"Default to silence between tool calls. Only write text when you find something, change direction, or hit a blocker — one sentence each. Do not narrate routine actions ('Now I'll...', 'Let me check...', 'Looking at...'). When done: one or two sentences on the outcome. Do not recap every file or test — the user has been following along."*

For knowledge-work deliverables (reports, analysis readouts), verbosity responds well to instructions in user preferences or the user turn — expose a verbosity preference rather than hard-coding a length.

**More deliberate — asks more often.** 4.8 is more deliberate than prior Opus models. On minor decisions it would previously just make (a variable name, a default value, which of two equivalent approaches), it tends to pause and ask, and often closes a completed task with "Want me to also…?" rather than doing the obvious next step or stopping cleanly. Preferred for high-stakes or unfamiliar codebases, but bugs users when uncalibrated. Grant autonomy on the small stuff while keeping caution where it matters (in Claude Code testing this cut ask-rate by ~12 percentage points with no increase in over-reach):

> *"For minor choices (naming, formatting, default values, which approach among equivalents), pick a reasonable option and note it rather than asking. For scope changes or destructive actions, still ask first."*

**Verbose reasoning when thinking is disabled.** With \`thinking: {type: "disabled"}\`, 4.8 occasionally writes longer explanations of its reasoning into the visible response, reading as verbose when the user wants a fast answer. The simplest fix is to leave adaptive thinking on — \`thinking: {type: "adaptive"}\` (the recommended setting; it adjusts how much to think per task). Adaptive is *not* on when the field is omitted — like Opus 4.7, a request with no \`thinking\` field runs without thinking, so set it explicitly. If you need thinking off for latency/cost, scope it in the system prompt:

> *"Respond only with your final answer. Do not include exploratory reasoning, intermediate drafts, diffs you considered but rejected, or meta-commentary about your process."*

### Opus 4.8 Migration Checklist

\`[BLOCKS]\` items cause a 400 if missed; \`[TUNE]\` items are quality/cost adjustments — surface as recommendations.

For a caller **already on Opus 4.7**, only the first item is required; everything else is \`[TUNE]\`. The conditional \`[BLOCKS]\` item applies only when coming from Opus 4.6 or earlier.

- [ ] **[BLOCKS]** Update the \`model=\` string to \`claude-opus-4-8\`
- [ ] **[BLOCKS]** *(only if coming from Opus 4.6 or earlier)* Apply the **Migrating to Opus 4.7** breaking changes first — \`budget_tokens\` → adaptive thinking, strip \`temperature\`/\`top_p\`/\`top_k\`, remove last-assistant-turn prefills. These already 400 on 4.7 and continue to 400 on 4.8.
- [ ] **[TUNE]** Long-horizon / agentic work: put the full task spec in one well-specified first turn and run at \`high\` or \`xhigh\` effort (Claude Code: \`/goal\`; Managed Agents: an Outcome with a gradeable rubric)
- [ ] **[TUNE]** Effort: sweep \`medium\` / \`high\` / \`xhigh\` on your eval set and pick per route by the intelligence ↔ latency ↔ cost tradeoff (default \`high\`, \`xhigh\` for coding/agentic)
- [ ] **[TUNE]** Research depth & tool use: add a search-first instruction; add explicit triggering guidance for subagents, file-based memory, and custom tools (4.8 under-reaches for these by default) — in the system prompt and in each tool's own \`description\` (prescriptive "call this when…" descriptions give measurable lift)
- [ ] **[TUNE]** Narration: remove forced-progress scaffolding (*"after every N tool calls…"*); add a silence-default if a coding agent is too chatty
- [ ] **[TUNE]** Autonomy: add small-decisions-don't-ask guidance to cut ask-rate, while keeping caution on scope changes / destructive actions
- [ ] **[TUNE]** Writing voice: re-evaluate style prompts added to counter 4.7's directness — 4.8 is warmer and less hedged by default
- [ ] **[TUNE]** Code-review harnesses: keep the report-everything-filter-downstream pattern (4.8 follows "only high-severity" / "be conservative" filters literally, which can depress measured recall)
- [ ] **[TUNE]** Thinking-disabled paths: add a final-answer-only instruction if reasoning leaks into the visible response
- [ ] **[TUNE]** Consider mid-session system messages (\`role:"system"\` in \`messages\`, beta \`mid-conversation-system-2026-04-07\`) for context the app learns mid-session, instead of rebuilding the top-level system prompt and invalidating the cache

---

## Verify the Migration

After updating, spot-check that the new model is actually being used. Replace \`YOUR_TARGET_MODEL\` with the model string you migrated to (e.g. \`claude-opus-4-8\`, \`claude-opus-4-7\`, \`claude-sonnet-4-6\`, \`claude-haiku-4-5\`) and keep the assertion prefix in sync:

\`\`\`python
YOUR_TARGET_MODEL = "{{OPUS_ID}}"  # or "claude-opus-4-7", "claude-sonnet-4-6", "claude-haiku-4-5"
response = client.messages.create(model=YOUR_TARGET_MODEL, max_tokens=64, messages=[...])
assert response.model.startswith(YOUR_TARGET_MODEL), response.model
\`\`\`

For rate-limit headroom changes, pricing, or capability deltas (vision, structured outputs, effort support), query the Models API:

\`\`\`python
m = client.models.retrieve(YOUR_TARGET_MODEL)
m.max_input_tokens, m.max_tokens
m.capabilities["effort"]["max"]["supported"]
\`\`\`

See \`shared/models.md\` for the full capability lookup pattern.
