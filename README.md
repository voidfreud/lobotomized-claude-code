# lobotomized-claude-code

<p align="center">
  <img src="./assets/banner.png" alt="lobotomized-claude-code" width="420">
</p>

System-prompt overrides for [Claude Code](https://claude.com/claude-code). Claude Code ships every model the same prompts — written for older Claudes and heavy with scaffolding, hedging, CAPS, and anti-laziness nagging that newer models don't need and often behave *worse* under. This repo cuts the bulk and rewrites the load-bearing parts in a register the model actually behaves better in.

[`skrabe/tweakcc-fixed`](https://github.com/skrabe/tweakcc-fixed) reads these `.md` files and patches your installed Claude Code in place.

## Which version do I use?

There are two prompt sets — pick the one matching your model:

| Your model | Use this folder |
|---|---|
| **Claude Fable 5** | [`system-prompts-fable-5/`](./system-prompts-fable-5) ← re-judged file-by-file against the Fable 5 / Mythos 5 system card and Anthropic's Fable prompting guide |
| **Claude Opus 4.8** | [`system-prompts-opus-4-8/`](./system-prompts-opus-4-8) ← reshaped from scratch against the Opus 4.8 system card |
| **Claude Opus 4.7 / 4.6** | [`system-prompts-opus-4-7/`](./system-prompts-opus-4-7) |

Not sure? Run `claude --version` and check which Opus you're on. The install below symlinks whichever set you choose; switching later is one `ln -sfn`.

## What you get (Fable 5 set)

| | Leaner by |
|---|---|
| **Always-on prompts** (system prompt + tool descriptions, inject every coding turn) | **~40%** |
| Whole catalogued corpus (371 prompts) | **~24%** |
| **37 prompts suppressed entirely** | unused features (Managed Agents, PowerShell, WSL), always-on upsells, duplicates |

The Fable set cuts *differently* than the 4.8 set, per the model's measured deltas:

- **Cut harder:** generic anti-injection boilerplate (Fable's injection resistance is ~2× better than 4.8 and classifier-backed), overrefusal hedging, moralizing, "be thorough" motivation, capability hand-holding, enumerated do/don't checklists, and terseness pressure — Fable over-compresses on its own; readable beats terse.
- **Kept and strengthened** (Fable's measured regressions vs 4.8): verify-before-claiming-done and report-failures-plainly, don't fabricate missing inputs, call mistakes mistakes, a thin irreversible-ops core plus don't-bypass-blockers, scope discipline, and persistence (no silent early-stop).
- **Restored to pristine** where Anthropic pre-tuned the prompt for Fable (outcome-first communication, autonomous-operation rules) — the old trims would have deleted content that now earns its place.
- **Reworded corpus-wide:** "show your thinking / explain your reasoning" phrasing, which can trip Fable's `reasoning_extraction` classifier and silently degrade the session to Opus 4.8.

## What you get (Opus 4.8 set)

| | Leaner by |
|---|---|
| **Always-on prompts** (inject every turn — harness, communication, doing-tasks, executing-actions, memory, core tools) | **~30%** |
| All behavior-shaping prompts | **~36%** |
| **39 prompts removed entirely** | unused features, duplicates, and safety theater |

Less always-on prompt means a faster first response, more headroom before compaction, and less drift from contradictory rules — without losing anything the model relies on.

### What it cuts vs. keeps

The Opus 4.8 set is shaped against the model's system card, not by taste:

- **Cut:** anti-laziness/"be thorough" scaffolding, CAPS theater (`MUST`/`NEVER`/`ALWAYS`), reflexive hedging and caveats, always-on upsells, restated rules, and feature docs you don't use — all things 4.8 either does by default or behaves worse under.
- **Kept (and sharpened):** the honesty contract (report failures truthfully, don't fake success), destructive-action confirmation, scope discipline, parallel-tool guidance, and exact output formats.
- **Strengthened:** on the one surface the card flags 4.8 as weaker — **browser / computer-use / web-fetch** — treat page and tool content as untrusted data, scrutinize intent, and confirm before destructive actions.

## Install

```bash
git clone https://github.com/skrabe/lobotomized-claude-code ~/.tweakcc/lobotomized-claude-code

# pick your set (Fable 5 shown; use system-prompts-opus-4-8 or -opus-4-7 for older models)
ln -sfn ~/.tweakcc/lobotomized-claude-code/system-prompts-fable-5 ~/.tweakcc/system-prompts
ln -sfn ~/.tweakcc/lobotomized-claude-code/system-reminders        ~/.tweakcc/system-reminders

# apply with the patcher
git clone https://github.com/skrabe/tweakcc-fixed ~/dev/tweakcc-fixed
cd ~/dev/tweakcc-fixed && pnpm install && pnpm build
node ~/dev/tweakcc-fixed/dist/index.mjs --apply
```

Re-run `node ~/dev/tweakcc-fixed/dist/index.mjs --apply` after each Claude Code update. To revert: `node ~/dev/tweakcc-fixed/dist/index.mjs --restore`.

> Use [`skrabe/tweakcc-fixed`](https://github.com/skrabe/tweakcc-fixed) (built from source, above) — not the `tweakcc` npm package. The skrabe fork carries the extraction + patching fixes these overrides depend on.

## License

MIT. Derivative overrides of Anthropic's Claude Code prompts. These change model behavior in non-trivial ways and aren't endorsed by Anthropic — use on your local install at your own risk.
