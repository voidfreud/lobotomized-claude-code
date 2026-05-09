# lobotomized-claude-code

<p align="center">
  <img src="./assets/banner.png" alt="lobotomized-claude-code" width="420">
</p>

System-prompt overrides for [Claude Code](https://claude.com/claude-code), tuned for **Claude Opus 4.7**.

CC ships every model the same prompt-by-volume Opus 4.6 needed. 4.7 follows instructions literally, gets jumpier on CAPS, and doesn't need the anti-laziness scaffolding. This repo cuts the bulk and rewrites the load-bearing fragments in a register the model actually behaves better under.

## ~67% off the daily flow. ~33% off everything.

| What you actually feel | Cut |
|---|---|
| **Daily flow** — the fragments that inject every interactive turn (harness, communication, doing-tasks, executing-actions, memory, always-loaded tool descriptions) | **~67% smaller** |
| Niche stuff — language-specific Anthropic API docs, model migration guide, calendar/cron skills, WSL settings, Windows tool descriptions | **~32% smaller** |

The first row is what matters. Niche prompts only inject when you trigger that feature (most users never do), so the more modest cut on those barely shows up in real sessions. **Every coding turn carries roughly 20K fewer characters of always-on prompt.** Faster first response, more headroom before compaction, less drift from contradictory always-on rules.

<details>
<summary>Methodology</summary>

Numbers measured against pristine CC 2.1.138 prompts. Anthropic doesn't publish an offline tokenizer for Claude 3+/4 models — accurate token counts require `client.messages.count_tokens()` against an API key — so figures are reported in **characters** (text length, no tokenization assumption). Same approximation isn't an issue when comparing two strings: the percentage cut is the percentage cut.

| | Files | Pristine | After lobotomy | Cut |
|---|---:|---:|---:|---:|
| Hot path (always or near-always injected) | ~24 | ~30K chars | ~10K chars | **−67%** |
| Cold path (conditional / niche) | ~267 | ~889K chars | ~608K chars | **−32%** |
| Total | 291 | ~919K chars | ~618K chars | **−33%** |

The cold-path bulk is reference content — `data-claude-api-reference-{python,typescript,go,java,csharp,php,ruby}`, the model migration guide, `data-managed-agents-*` API docs, the `skill-*` cron tasks (catch-up, dream, morning-checkin, pre-meeting). None of it injects in normal coding sessions, which is why the headline focuses on the daily-flow row.
</details>

## What this aims at

Six things, all from the [Opus 4.7 prompting guide](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-7-best-practices):

1. **Less is more.** 4.7 is more literal — fewer rules, less drift.
2. **No CAPS theater.** `STRICTLY PROHIBITED` / `CRITICAL REQUIREMENT` / `MUST` triggers overcorrection on 4.7. Plain directives behave better.
3. **Parallel by default.** The harness now explicitly tells the model: independent tool calls go in one message, multiple blocks. Reading 5 files = 5 parallel reads. Stop running things in series when they don't have to be.
4. **Grill-me on plan mode.** Plan mode Phase 1 is now [Matt Pocock's grill-me pattern](https://www.aihero.dev/my-grill-me-skill-has-gone-viral) — the model interviews you one decision at a time, recommends an answer, walks the design tree before producing a plan. Underspecified plans are more expensive than the questions.
5. **No always-on CTAs.** Killed the `/schedule` upsell prompt. Things 4.7 follows literally shouldn't be marketing copy.
6. **Tighter destructive-action guards.** Git commit/push/merge/PR all require explicit confirmation in the current conversation. "Commit and push" splits into two confirmations. The model can't make your DB go away because it got stuck.

Inspirations: [pi](https://github.com/earendil-works/pi/tree/main/packages/coding-agent) and Factory Droid both ship full coding-agent system prompts in roughly the size of a single CC tool description. They work fine.

## How it works

Each `.md` in [`system-prompts/`](./system-prompts) is an override fragment with frontmatter naming the original prompt and the CC version it targets. [`tweakcc-fixed`](https://github.com/skrabe/tweakcc-fixed) reads these and patches the `cli.js` (or native binary) of your installed Claude Code in place.

Load-bearing structure (variable substitutions like `${ASK_USER_QUESTION_TOOL_NAME}`, conditional blocks for env flags) is preserved verbatim. Anywhere you see `${...}`, that's CC injecting runtime values — leaving them alone is non-negotiable.

## Install

```bash
git clone https://github.com/skrabe/lobotomized-claude-code ~/.tweakcc/lobotomized-claude-code
mv ~/.tweakcc/system-prompts ~/.tweakcc/system-prompts.bak 2>/dev/null
ln -sfn ~/.tweakcc/lobotomized-claude-code/system-prompts ~/.tweakcc/system-prompts

git clone https://github.com/skrabe/tweakcc-fixed ~/dev/tweakcc-fixed
cd ~/dev/tweakcc-fixed && pnpm install && pnpm build
node ~/dev/tweakcc-fixed/dist/index.mjs --apply
```

Re-run `node ~/dev/tweakcc-fixed/dist/index.mjs --apply` after every Claude Code update.

`skrabe/tweakcc-fixed` is a [direct fork](https://github.com/skrabe/tweakcc-fixed) of [Piebald-AI/tweakcc](https://github.com/Piebald-AI/tweakcc) carrying unmerged patches that make `--apply` work on current CC versions. The `tweakcc-fixed` package on npm is a different fork ([BenIsLegit/tweakcc-fixed](https://github.com/BenIsLegit/tweakcc-fixed)) — these overrides are tuned against `skrabe/tweakcc-fixed` HEAD, so install from source above rather than via `npx`.

## License

MIT. Derivative overrides of Anthropic's Claude Code prompts. Use on your local install at your own risk — these change the model's behavior in non-trivial ways and aren't endorsed by Anthropic.
