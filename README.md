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
| **Claude Opus 4.8** | [`system-prompts-opus-4-8/`](./system-prompts-opus-4-8) ← reshaped from scratch against the Opus 4.8 system card |
| **Claude Opus 4.7 / 4.6** | [`system-prompts-opus-4-7/`](./system-prompts-opus-4-7) |

Not sure? Run `claude --version` and check which Opus you're on. The install below symlinks whichever set you choose; switching later is one `ln -sfn`.

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

# pick your set (Opus 4.8 shown; use system-prompts-opus-4-7 for 4.7/4.6)
ln -sfn ~/.tweakcc/lobotomized-claude-code/system-prompts-opus-4-8 ~/.tweakcc/system-prompts
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
