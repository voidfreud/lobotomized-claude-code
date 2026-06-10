<!--
name: 'Skill: Building LLM-powered applications with Claude'
description: >-
  Guides Claude in building LLM-powered applications using the Anthropic SDK,
  covering language detection, API surface selection (Claude API vs Managed
  Agents), model defaults, thinking/effort configuration, and language-specific
  documentation reading
ccVersion: 2.1.170
-->
# Building LLM-Powered Applications with Claude

Choose the right surface, detect the project language, then read the relevant language-specific documentation.

## Before You Start

Scan the target file (or, if none, the prompt and project) for non-Anthropic provider markers — `import openai`, `from openai`, `langchain_openai`, `OpenAI(`, `gpt-4`, `gpt-5`, file names like `agent-openai.py` or `*-generic.py`, or any instruction to keep the code provider-neutral. If you find any, stop and tell the user this skill produces Claude/Anthropic SDK code; ask whether to switch the file to Claude or write a non-Claude implementation. Do not put Anthropic SDK calls into a non-Anthropic file.

## Output Requirement

When asked to add, modify, or implement a Claude feature, call Claude through one of:

1. **The official Anthropic SDK** for the project's language (`anthropic`, `@anthropic-ai/sdk`, `com.anthropic.*`, etc.). Default whenever a supported SDK exists.
2. **Raw HTTP** (`curl`, `requests`, `fetch`, `httpx`, etc.) — only when the user explicitly asks for cURL/REST/raw HTTP, the project is a shell/cURL project, or the language has no official SDK.

Don't mix the two, and don't fall back to OpenAI-compatible shims.

Don't guess SDK usage. Function names, class names, namespaces, method signatures, and import paths must come from explicit documentation — the `{lang}/` files in this skill, or the official SDK repos/docs in `shared/live-sources.md`. If the binding you need isn't documented in the skill files, WebFetch the relevant SDK repo from `shared/live-sources.md` before writing code. Do not infer Ruby/Java/Go/PHP/C# APIs from cURL shapes or another language's SDK.

## Defaults

Unless the user requests otherwise: use {{OPUS_NAME}} — model string `{{OPUS_ID}}`. Default to adaptive thinking (`thinking: {type: "adaptive"}`) for anything non-trivial. Default to streaming for long input/output or high `max_tokens` (avoids request timeouts) — use `.get_final_message()` / `.finalMessage()` if you don't need per-event handling.

---

## Subcommands

If the User Request at the bottom is a bare subcommand string (no prose), search every **Subcommands** table in this document — including any in appended sections — and follow the matching Action column. This lets users invoke flows via `/claude-api <subcommand>`. If no table matches, treat the request as normal prose.

| Subcommand | Action |
|---|---|
| `migrate` | Migrate existing Claude API code to a newer model. Read `shared/model-migration.md` and follow it in order: Step 0 (confirm scope — ask which files/directories before any edit), Step 1 (classify each file), then the per-target breaking-changes section. Execute it, don't summarize. If the user didn't name a target model, ask which model to migrate to in the same turn as the scope question. |

---

## Language Detection

Before reading code examples, determine the language:

1. **Infer from project files:**

   - `*.py`, `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` → **Python** — read from `python/`
   - `*.ts`, `*.tsx`, `package.json`, `tsconfig.json` → **TypeScript** — read from `typescript/`
   - `*.js`, `*.jsx` (no `.ts` present) → **TypeScript** — JS uses the same SDK, read from `typescript/`
   - `*.java`, `pom.xml`, `build.gradle` → **Java** — read from `java/`
   - `*.kt`, `*.kts`, `build.gradle.kts` → **Java** — Kotlin uses the Java SDK, read from `java/`
   - `*.scala`, `build.sbt` → **Java** — Scala uses the Java SDK, read from `java/`
   - `*.go`, `go.mod` → **Go** — read from `go/`
   - `*.rb`, `Gemfile` → **Ruby** — read from `ruby/`
   - `*.cs`, `*.csproj` → **C#** — read from `csharp/`
   - `*.php`, `composer.json` → **PHP** — read from `php/`

2. **Multiple languages detected:** check which the user's current file or question relates to; if still ambiguous, ask which language they're using for the Claude API integration.

3. **Can't infer** (empty project, no source, unsupported language): use AskUserQuestion with options Python, TypeScript, Java, Go, Ruby, cURL/raw HTTP, C#, PHP. If AskUserQuestion is unavailable, default to Python and say so.

4. **Unsupported language** (Rust, Swift, C++, Elixir, etc.): suggest cURL/raw HTTP from `curl/`, note community SDKs may exist, offer Python or TypeScript as reference.

5. **User wants cURL/raw HTTP:** read from `curl/`.

### Language-Specific Feature Support

| Language   | Tool Runner | Managed Agents | Notes                                 |
| ---------- | ----------- | -------------- | ------------------------------------- |
| Python     | Yes (beta)  | Yes (beta)     | Full support — `@beta_tool` decorator |
| TypeScript | Yes (beta)  | Yes (beta)     | Full support — `betaZodTool` + Zod    |
| Java       | Yes (beta)  | Yes (beta)     | Beta tool use with annotated classes  |
| Go         | Yes (beta)  | Yes (beta)     | `BetaToolRunner` in `toolrunner` pkg  |
| Ruby       | Yes (beta)  | Yes (beta)     | `BaseTool` + `tool_runner` in beta    |
| C#         | Yes (beta)  | Yes (beta)     | `BetaToolRunner` + raw JSON schema    |
| PHP        | Yes (beta)  | Yes (beta)     | `BetaRunnableTool` + `toolRunner()`   |
| cURL       | N/A         | Yes (beta)     | Raw HTTP, no SDK features             |

> **Managed Agents code examples:** read your language's `{lang}/managed-agents/README.md` (or `curl/managed-agents.md`) plus the language-agnostic `shared/managed-agents-*.md` concept files. C# uses `client.Beta.Agents` and related namespaces. **Agents are persistent — create once, reference by ID** (see the Managed Agents section below).

---

## Which Surface Should I Use?

> **Start simple.** Single calls and workflows handle most cases. Reach for agents only when the task genuinely requires open-ended, model-driven exploration.

| Use Case                                        | Tier            | Recommended Surface       | Why                                                          |
| ----------------------------------------------- | --------------- | ------------------------- | ------------------------------------------------------------ |
| Classification, summarization, extraction, Q&A  | Single LLM call | **Claude API**            | One request, one response                                    |
| Batch processing or embeddings                  | Single LLM call | **Claude API**            | Specialized endpoints                                        |
| Multi-step pipelines with code-controlled logic | Workflow        | **Claude API + tool use** | You orchestrate the loop                                     |
| Custom agent with your own tools                | Agent           | **Claude API + tool use** | Maximum flexibility                                          |
| Server-managed stateful agent with workspace    | Agent           | **Managed Agents**        | Anthropic runs the loop and hosts the tool-execution sandbox |
| Persisted, versioned agent configs              | Agent           | **Managed Agents**        | Agents are stored objects; sessions pin to a version         |
| Long-running multi-turn agent with file mounts  | Agent           | **Managed Agents**        | Per-session containers, SSE event stream, Skills + MCP       |

> Use Managed Agents when you want Anthropic to run the agent loop *and* host the container where tools execute (file ops, bash, code execution all run in the per-session workspace). Use Claude API + tool use when you host the compute or run a custom tool runtime — tool runner for automatic looping, or the manual loop for fine-grained control (approval gates, custom logging, conditional execution).

> **Cloud-provider access.** **Claude Platform on AWS** is Anthropic-operated with same-day API parity — Managed Agents and every feature here work there **except self-hosted sandboxes** (see `shared/claude-platform-on-aws.md`). **Amazon Bedrock**, **Google Vertex AI**, and **Microsoft Foundry** do **not** support Managed Agents or Anthropic server-side tools; use **Claude API + tool use** there.

### Decision Tree

```
What does your application need?

0. Which provider?
   ├── First-party API or Claude Platform on AWS → continue (full surface available).
   └── Amazon Bedrock, Google Vertex AI, or Microsoft Foundry → Claude API (+ tool use for agents); Managed Agents not available there.

1. Single LLM call (classification, summarization, extraction, Q&A)
   └── Claude API — one request, one response

2. Want Anthropic to run the agent loop and host a per-session
   container where Claude executes tools (bash, file ops, code)?
   └── Yes → Managed Agents — server-managed sessions, persisted agent configs,
       SSE event stream, Skills + MCP, file mounts.

3. Workflow (multi-step, code-orchestrated, with your own tools)
   └── Claude API with tool use — you control the loop

4. Open-ended agent (model decides its trajectory, your own tools, you host the compute)
   └── Claude API agentic loop (maximum flexibility)
```

### Should I Build an Agent?

Before the agent tier, check all four:

- **Complexity** — multi-step and hard to fully specify in advance? ("turn this design doc into a PR" vs. "extract the title from this PDF")
- **Value** — does the outcome justify higher cost and latency?
- **Viability** — is Claude capable at this task type?
- **Cost of error** — can errors be caught and recovered (tests, review, rollback)?

If any answer is "no", stay at a simpler tier.

---

## Architecture

Everything goes through `POST /v1/messages`. Tools and output constraints are features of this single endpoint, not separate APIs.

**User-defined tools** — you define tools (decorators, Zod schemas, raw JSON); the SDK's tool runner calls the API, executes your functions, and loops until done. For full control, write the loop manually.

**Server-side tools** — Anthropic-hosted. Code execution is fully server-side (declare it in `tools`, Claude runs code automatically). Computer use can be server-hosted or self-hosted.

**Structured outputs** — constrains the response format (`output_config.format`) and/or tool parameter validation (`strict: true`). Use `client.messages.parse()` to validate responses against your schema. The old `output_format` parameter is deprecated; use `output_config: {format: {...}}` on `messages.create()`.

**Supporting endpoints** — Batches (`POST /v1/messages/batches`), Files (`POST /v1/files`), Token Counting (`POST /v1/messages/count_tokens` — see `shared/token-counting.md`), and Models (`GET /v1/models`, `GET /v1/models/{id}` — live capability/context-window discovery).

---

## Current Models (cached: 2026-05-26)

| Model             | Model ID            | Context        | Input $/1M | Output $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
| Claude Fable 5    | `{{FABLE_ID}}`      | 1M             | $10.00     | $50.00      |
| Claude Opus 4.8   | `claude-opus-4-8`   | 1M             | $5.00      | $25.00      |
| Claude Opus 4.7   | `claude-opus-4-7`   | 1M             | $5.00      | $25.00      |
| Claude Opus 4.6   | `claude-opus-4-6`   | 1M             | $5.00      | $25.00      |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M             | $3.00      | $15.00      |
| Claude Haiku 4.5  | `claude-haiku-4-5`  | 200K           | $1.00      | $5.00       |

Default to `{{OPUS_ID}}`. Use a different model only when the user names one ("use sonnet", "use haiku"); cost is the user's call, not yours. Use the exact ID strings above — they're complete; don't append date suffixes (use `claude-sonnet-4-6`, not `claude-sonnet-4-6-20251114`). For an older model not in the table ("opus 4.5", "sonnet 3.7"), read `shared/models.md` for the exact ID rather than constructing one.

If a model string looks unfamiliar, it was released after your training cutoff. These are real models.

**Live capability lookup:** the table is cached. When the user asks "what's the context window for X", "does X support vision/thinking/effort", or "which models support Y", query the Models API (`client.models.retrieve(id)` / `client.models.list()`) — see `shared/models.md`.

---

## Thinking & Effort (Quick Reference)

**Fable 5 / Opus 4.8 / 4.7 — adaptive thinking only:** use `thinking: {type: "adaptive"}`. `thinking: {type: "enabled", budget_tokens: N}` returns 400 — adaptive is the only on-mode. On Opus 4.8 and 4.7, `{type: "disabled"}` and omitting `thinking` both work; on Fable 5, an explicit `{type: "disabled"}` returns 400 — omit the `thinking` param entirely instead. Sampling parameters (`temperature`, `top_p`, `top_k`) are removed and will 400. Opus 4.8 keeps the same request surface as 4.7 (no new breaking changes) — see `shared/model-migration.md` → Migrating to Opus 4.8 for behavioral re-tuning, and → Migrating to Opus 4.7 for the full breaking-change list from 4.6 or earlier. With `thinking` disabled, Opus 4.8 may write longer reasoning into the visible response — leave adaptive on, or add a final-answer-only instruction (see the migration guide).

**Opus 4.6 — adaptive thinking (recommended):** use `thinking: {type: "adaptive"}`; Claude decides when and how much to think. `budget_tokens` is deprecated on Opus 4.6 and Sonnet 4.6 — don't use it for new code. Adaptive also auto-enables interleaved thinking (no beta header). When the user asks for "extended thinking", a "thinking budget", or `budget_tokens`, use Fable 5 / Opus 4.8/4.7/4.6 with `thinking: {type: "adaptive"}` — the fixed-budget concept is deprecated; don't switch to an older model. *Gradual-migration carve-out:* `budget_tokens` is still functional on Opus 4.6 and Sonnet 4.6 as a transitional escape hatch — see `shared/model-migration.md` → Transitional escape hatch. This carve-out does **not** apply to Fable 5, Opus 4.7 or 4.8 — `budget_tokens` is fully removed there.

**Effort (GA, no beta header):** controls thinking depth and overall token spend via `output_config: {effort: "low"|"medium"|"high"|"max"}` (inside `output_config`, not top-level). Default is `high` (= omitting it). `max` is supported on Fable 5, Opus 4.6+, and Sonnet 4.6 (not Haiku or earlier Sonnets). Opus 4.7 added `"xhigh"` (between `high` and `max`) — best for most coding/agentic use on Fable 5 / Opus 4.7/4.8 and the default in Claude Code; use a minimum of `high` for intelligence-sensitive work. Works on Fable 5, Opus 4.5/4.6/4.7/4.8 and Sonnet 4.6; errors on Sonnet 4.5 / Haiku 4.5. On Fable 5 / Opus 4.7/4.8 effort matters more than on any prior Opus — re-tune when migrating, and run long-horizon/agentic tasks at `high`/`xhigh` with the full task spec up front. Lower effort means fewer, more-consolidated tool calls and terser confirmations. Use `max` when correctness matters more than cost; `low` for subagents or simple tasks.

**Fable 5 / Opus 4.8 / 4.7 — thinking content omitted by default:** `thinking` blocks stream but their text is empty unless you opt in with `thinking: {type: "adaptive", display: "summarized"}` (default `"omitted"`). Silent — no error. If you stream reasoning to users, the default looks like a long pause; set `"summarized"` to restore visible progress.

**Task Budgets (beta, Fable 5 / Opus 4.7 / 4.8):** `output_config: {task_budget: {type: "tokens", total: N}}` tells the model its token allowance for a full agentic loop — it sees a running countdown and self-moderates (minimum 20,000; beta header `task-budgets-2026-03-13`). Distinct from `max_tokens`, an enforced per-response ceiling the model isn't aware of. See `shared/model-migration.md` → Task Budgets.

**Sonnet 4.6:** supports adaptive thinking; `budget_tokens` deprecated — use adaptive.

**Older models (only if explicitly requested):** for Sonnet 4.5 or another older model, use `thinking: {type: "enabled", budget_tokens: N}` where `budget_tokens` < `max_tokens` (minimum 1024). Don't pick an older model just because the user mentions `budget_tokens`.

---

## Compaction (Quick Reference)

**Beta, Fable 5 / Opus 4.8/4.7/4.6 and Sonnet 4.6.** For conversations that may exceed 1M context, enable server-side compaction: the API summarizes earlier context as it nears the trigger threshold (default 150K tokens). Beta header `compact-2026-01-12`.

Append `response.content` (not just the text) back to your messages every turn. Compaction blocks must be preserved — the API uses them to replace compacted history on the next request; appending only the text string silently loses compaction state.

See `{lang}/claude-api/README.md` (Compaction section) for code examples; full docs via WebFetch in `shared/live-sources.md`.

---

## Prompt Caching (Quick Reference)

**Prefix match.** Any byte change in the prefix invalidates everything after it. Render order is `tools` → `system` → `messages`. Keep stable content first (frozen system prompt, deterministic tool list); put volatile content (timestamps, per-request IDs, varying questions) after the last `cache_control` breakpoint.

**Mid-conversation operator instructions** (beta header `mid-conversation-system-2026-04-07`, on supporting models): append `{"role": "system", ...}` to `messages[]` instead of editing top-level `system`. Preserves the cached prefix and is the prompt-injection-safe operator channel. See `shared/prompt-caching.md` § Mid-conversation system messages.

**Top-level auto-caching** (`cache_control: {type: "ephemeral"}` on `messages.create()`) is simplest when you don't need fine-grained placement. Max 4 breakpoints per request. Minimum cacheable prefix is ~1024 tokens — shorter prefixes silently won't cache.

**Verify with `usage.cache_read_input_tokens`** — if it's zero across repeated requests, a silent invalidator is at work (`datetime.now()` in system prompt, unsorted JSON, varying tool set).

For placement patterns, architectural guidance, and the silent-invalidator audit checklist: read `shared/prompt-caching.md`. Language-specific syntax: `{lang}/claude-api/README.md` (Prompt Caching section).

---

## Managed Agents (Beta)

A third surface: server-managed stateful agents with Anthropic-hosted tool execution. Create a persisted, versioned Agent config (`POST /v1/agents`), then start Sessions that reference it. Each session provisions a container as the agent's workspace — bash, file ops, code execution run there; the agent loop runs on Anthropic's orchestration layer and acts on the container via tools. The session streams events; you send messages and tool results back.

Available on the first-party API and Claude Platform on AWS. **Not** available on Amazon Bedrock, Google Vertex AI, or Microsoft Foundry — use Claude API + tool use there.

**Mandatory flow:** Agent (once) → Session (every run). `model`/`system`/`tools` live on the agent, never the session. See `shared/managed-agents-overview.md` for the full reading guide, beta headers, and pitfalls.

**Beta headers:** `managed-agents-2026-04-01` — the SDK sets this automatically for all `client.beta.{agents,environments,sessions,vaults,memory_stores}.*` calls. Skills API uses `skills-2025-10-02` and Files API uses `files-api-2025-04-14`, passed automatically except for `/v1/skills` and `/v1/files`.

**Subcommands** — invoke with `/claude-api <subcommand>`:

| Subcommand | Action |
|---|---|
| `managed-agents-onboard` | Walk the user through setting up a Managed Agent from scratch. Read `shared/managed-agents-onboarding.md` and run its interview script: mental model → know-or-explore branch → template config → session setup → **pre-flight viability check** → emit code. The viability check (reconcile the stated job against configured tools/credentials/data) catches under-resourced setups before the agent burns budget. Run the interview, don't summarize. |

**Reading guide:** start with `shared/managed-agents-overview.md`, then the topical `shared/managed-agents-*.md` files (core, environments, tools, events, outcomes, multiagent, webhooks, memory, client-patterns, onboarding, api-reference). For Python, TypeScript, Go, Ruby, PHP, and Java read `{lang}/managed-agents/README.md`; for cURL read `curl/managed-agents.md`. **Agents are persistent — create once, reference by ID.** Store the agent ID returned by `agents.create` and pass it to every subsequent `sessions.create`; don't call `agents.create` in the request path. The Anthropic CLI (`ant`) creates agents and environments from version-controlled YAML — see `shared/anthropic-cli.md`. If a binding isn't shown in the README, WebFetch the relevant entry from `shared/live-sources.md`. C# uses `client.Beta.Agents`.

**When the user wants to set up a Managed Agent from scratch** ("how do I get started", "walk me through creating one", "set up a new agent"): read `shared/managed-agents-onboarding.md` and run its interview — same flow as `managed-agents-onboard`.

**When the user asks "how do I write the client code for X":** read `shared/managed-agents-client-patterns.md` — lossless stream reconnect, `processed_at` queued/processed gate, interrupt, `tool_confirmation` round-trip, the idle/terminated break gate, post-idle status race, stream-first ordering, file-mount gotchas, keeping credentials host-side via custom tools, etc.

---

## Reading Guide

After detecting the language, read based on what the user needs.

### Quick Task Reference

**Single text classification/summarization/extraction/Q&A:** `{lang}/claude-api/README.md`
**Chat UI or real-time response display:** `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`
**Long-running conversations (may exceed context window):** `{lang}/claude-api/README.md` — Compaction section
**Migrating to a newer model or replacing a retired one:** `shared/model-migration.md`
**Prompt caching / "why is my cache hit rate low":** `shared/prompt-caching.md` + `{lang}/claude-api/README.md` (Prompt Caching section)
**Count tokens in a file / prompt / diff ("how many tokens is X"):** `shared/token-counting.md` — use `messages.count_tokens`, never `tiktoken`
**Function calling / tool use / agents:** `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`
**Agent design (tool surface, context management, caching strategy):** `shared/agent-design.md`
**Batch processing (non-latency-sensitive):** `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`
**File uploads across multiple requests:** `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`
**Managed Agents:** `shared/managed-agents-overview.md` + the other `shared/managed-agents-*.md` files, plus `{lang}/managed-agents/README.md` (or `curl/managed-agents.md`). See the Managed Agents section above for the persistence and reading details.

### Claude API (Full File Reference)

Read the language-specific folder (`{language}/claude-api/`):

1. **`{language}/claude-api/README.md`** — read first. Installation, quick start, common patterns, error handling.
2. **`shared/tool-use-concepts.md`** — function calling, code execution, memory, structured outputs (concepts).
3. **`shared/agent-design.md`** — designing an agent: bash vs. dedicated tools, programmatic tool calling, tool search/skills, context editing vs. compaction vs. memory, caching principles.
4. **`{language}/claude-api/tool-use.md`** — language-specific tool use examples (tool runner, manual loop, code execution, memory, structured outputs).
5. **`{language}/claude-api/streaming.md`** — chat UIs and incremental display.
6. **`{language}/claude-api/batches.md`** — many offline requests; runs asynchronously at 50% cost.
7. **`{language}/claude-api/files-api.md`** — sending the same file across multiple requests.
8. **`shared/prompt-caching.md`** — adding or optimizing caching; prefix stability, breakpoint placement, silent-invalidator anti-patterns.
9. **`shared/error-codes.md`** — debugging HTTP errors / error handling.
10. **`shared/model-migration.md`** — upgrading models, replacing retired models, translating `budget_tokens`/prefill patterns.
11. **`shared/live-sources.md`** — WebFetch URLs for the latest official docs.

> For Java, Go, Ruby, C#, PHP, and cURL — a single file each covers the basics; read that plus `shared/tool-use-concepts.md` and `shared/error-codes.md` as needed.

---

## When to Use WebFetch

Use WebFetch for the latest documentation when the user asks for "latest"/"current" info, cached data seems incorrect, or they ask about features not covered here. Live URLs are in `shared/live-sources.md`.

## Common Pitfalls

- Don't truncate inputs when passing files or content. If content exceeds the context window, notify the user and discuss options (chunking, summarization) rather than silently truncating.
- **Fable 5 / Opus 4.8 / 4.7 thinking:** adaptive only. `thinking: {type: "enabled", budget_tokens: N}` returns 400 — `budget_tokens` is fully removed (with `temperature`, `top_p`, `top_k`). Use `thinking: {type: "adaptive"}`. 4.8 inherits this surface from 4.7 with no new breaking changes; Fable 5 adds one — an explicit `thinking: {type: "disabled"}` returns 400 (accepted on 4.7/4.8), so omit the param instead.
- **Opus 4.6 / Sonnet 4.6 thinking:** use `thinking: {type: "adaptive"}` — don't use `budget_tokens` for new code (deprecated on both; transitional escape hatch in `shared/model-migration.md` doesn't apply to Fable 5 / 4.7/4.8). For older models, `budget_tokens` < `max_tokens` (minimum 1024), or it errors.
- **Prefill removed (Fable 5 and the 4.6/4.7/4.8 family):** last-assistant-turn prefills return 400 on Fable 5, Opus 4.6/4.7/4.8 and Sonnet 4.6. Use structured outputs (`output_config.format`) or system-prompt instructions to control format.
- **Confirm migration scope before editing:** when asked to migrate without a named file/directory/list, ask which scope first (whole working dir, a subdirectory, or specific files). Don't edit until confirmed. "migrate my codebase", "upgrade to Sonnet 4.6", bare "migrate to Opus 4.8" are still ambiguous — they say what, not where. Proceed without asking only when an exact file/directory/list is named. See `shared/model-migration.md` Step 0.
- **`max_tokens` defaults:** don't lowball it — hitting the cap truncates mid-thought. Non-streaming: default `~16000` (under SDK HTTP timeouts). Streaming: default `~64000`. Go lower only with a reason: classification (`~256`), cost caps, deliberately short outputs, or `max_tokens: 0` for cache pre-warming (see `shared/prompt-caching.md` → Pre-warming).
- **128K output tokens:** Fable 5, Opus 4.6/4.7/4.8 support up to 128K `max_tokens`, but the SDKs require streaming for values that large. Use `.stream()` with `.get_final_message()` / `.finalMessage()`.
- **Tool call JSON parsing (Fable 5 and the 4.6/4.7/4.8 family):** these may produce different JSON string escaping in tool-call `input` (Unicode, forward-slash). Always parse with `json.loads()` / `JSON.parse()` — never raw string matching on the serialized input.
- **Structured outputs (all models):** use `output_config: {format: {...}}`, not the deprecated `output_format` parameter on `messages.create()`.
- **Don't reimplement SDK functionality:** use `stream.finalMessage()` instead of wrapping `.on()` events in `new Promise()`; use typed exception classes (`Anthropic.RateLimitError`, etc.) instead of string-matching errors; use SDK types (`Anthropic.MessageParam`, `Anthropic.Tool`, `Anthropic.Message`, etc.) instead of redefining interfaces.
- **Report and document output:** for reports/documents/visualizations, the code execution sandbox has `python-docx`, `python-pptx`, `matplotlib`, `pillow`, and `pypdf` pre-installed. Claude can generate formatted files (DOCX, PDF, charts) and return them via the Files API — prefer this over plain stdout text for "report"/"document" requests.
