# Working in this repo

You're a coding agent invoked in this repo. Read this first. It explains what we're trying to do, where the moving parts live, and the verification rules that prevent the most common breakages.

## What this is

`lobotomized-claude-code` is a set of system-prompt overrides for [Claude Code](https://claude.com/claude-code), tuned for **Claude Opus 4.7**. Each `.md` in [`system-prompts/`](./system-prompts) replaces one of CC's built-in prompt fragments. A separate tool ([`tweakcc-fixed`](https://github.com/skrabe/tweakcc-fixed), see below) reads these files and patches the user's installed CC binary in place.

## What we're trying to achieve

CC ships every model the same prompt-by-volume that worked for older Claudes. Opus 4.7 follows instructions more literally, overtriggers on CAPS, and doesn't need the anti-laziness scaffolding. **The goal is to cut bulk and rewrite load-bearing prompts in a register the model behaves better under.**

Six concrete principles, all from the [Opus 4.7 prompting guide](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-7-best-practices). Read it before changing prompts:

1. **Less is more.** 4.7 is more literal — fewer rules, less drift. If you can cut a sentence without losing information, cut it.
2. **No CAPS theater.** `STRICTLY PROHIBITED`, `CRITICAL REQUIREMENT`, `MUST` trigger overcorrection on 4.7. Use plain directives.
3. **Parallel by default.** Independent tool calls go in one message, multiple blocks. The harness prompt explicitly says so.
4. **Interview before planning.** Plan mode runs Matt Pocock's [grill-me](https://www.aihero.dev/my-grill-me-skill-has-gone-viral) pattern in Phase 1: walk the design tree one decision at a time with a recommended answer.
5. **No always-on CTAs.** 4.7 follows literal CTAs; an "end every reply with /schedule" prompt becomes spam. Cull always-on upsells.
6. **Tighter destructive-action guards.** Git commit/push/merge/PR all require explicit confirmation in the current conversation. "Commit and push" splits into two confirmations.

For migrating prompts when Anthropic releases a new model (Opus 4.6 → 4.7, etc.), the [migration guide](https://docs.claude.com/en/docs/about-claude/models/migration-guide) is canonical. It documents breaking changes (`budget_tokens` → adaptive thinking, prefilled responses removed, sampling params removed on 4.7), silent default changes, and prompt-behavior shifts.

For comparison points on what minimal coding-agent prompts look like:
- [pi-mono coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) — main system prompt is ~50 lines including all tool descriptions
- Factory Droid — ~800 tokens for the full interactive system prompt (extracted via [tweakdroid](https://github.com/skrabe/tweakdroid))

## Repo layout

```
~/.tweakcc/lobotomized-claude-code/        ← THIS repo (canonical, has .git, GitHub remote skrabe/lobotomized-claude-code)
~/.tweakcc/system-prompts                  ← symlink → ./system-prompts (tweakcc-fixed reads from here)
~/dev/lobotomized-claude-code/             ← parallel clone for dev workflow

~/dev/tweakcc-fixed/                       ← the patcher (skrabe/tweakcc-fixed)
~/.tweakcc/config.json                     ← user's tweakcc settings (toggles, themes, etc.)
~/.tweakcc/orphans-removed-for-X.Y.Z/      ← prompts archived because the binary no longer references them
~/.tweakcc/native-binary.backup            ← pristine CC binary (auto-saved before first patch)
```

Each `.md` in `system-prompts/` has frontmatter:

```markdown
<!--
name: 'Tool Description: Edit'
description: ...
ccVersion: 2.1.128
variables:
  - MUST_READ_FIRST_FN
  - ADDITIONAL_EDIT_GUIDELINES_NOTE
-->
<the override content; ${VAR} interpolations referenced from variables: above>
```

The `variables:` list is metadata; the actual binding happens via the pristine prompt's `identifierMap` in `tweakcc-fixed/data/prompts/prompts-X.Y.Z.json`.

## How it works (just enough to reason about it)

1. CC's binary stores each prompt as a JS template literal containing `${MINIFIED_NAME}` interpolations. Anthropic ships `data/prompts/prompts-X.Y.Z.json` mirrors of these prompts with human-readable identifier names (the `identifierMap`).
2. `tweakcc-fixed --apply` reads each `.md` override, builds a regex from the pristine prompt pieces (with placeholders for the minified vars), finds the match in CC's `cli.js`, captures the actual minified names, and replaces the matched span with the user's override content (substituting captured names back into `${VAR}` references).
3. Native installations: `cli.js` is embedded in a Bun-compiled binary. tweakcc-fixed extracts via `node-lief`, patches, and repacks.
4. `--apply` runs through `~/.tweakcc/config.json` toggles for the non-prompt patches (max-effort default, model customizations, themes, etc.) too.

## Tweakcc-fixed status, and the fork-hop problem

`tweakcc-fixed` is `BenIsLegit/tweakcc-fixed` — a fork of `Piebald-AI/tweakcc` (the upstream maintained by Piebald) carrying ~25 fix commits Ben hadn't gotten upstream-merged yet. **Ben hasn't pushed since 2026-04-22**, while upstream has shipped 10 prompt-version drops (2.1.117 → 2.1.128). At the user's pace, Ben is effectively unmaintained — they're now upstream-of-stale-fork-of-upstream.

The user's working fork is `skrabe/tweakcc-fixed`, which is downstream of Ben. The remote layout in `~/dev/tweakcc-fixed`:

```
ben       https://github.com/BenIsLegit/tweakcc-fixed   (the stale fork-of-upstream)
origin    https://github.com/skrabe/tweakcc-fixed       (user's push target)
upstream  https://github.com/Piebald-AI/tweakcc         (Piebald — actively maintained)
```

**Migration the user is considering: re-fork directly off `Piebald-AI/tweakcc`, cherry-pick only the still-useful Ben commits, then maintain that fork directly.** Less friction, no dead intermediary. Ben's commits worth preserving (still relevant on current CC):

- `c87898c` — `verbose:X` not replaced inside destructure patterns
- `3114c5b`, `dc84a6c`, `89555eb`, `cc12f96`, `c66f604`, `9ef9328`, `b963ebf`, `f89a998` — userMessageDisplay bg/wrap/[object Object]/native-Box-attrs fixes
- `1e28b59` — thinkingVerbs past-tense regex for shrunken `-ed` array
- `f6ac8f3` — migration test cleanup
- TS7 build / Linux native-binary patching commits already on `skrabe/tweakcc-fixed`

Some Ben commits are skippable (branding rebumps, version bumps tied to the `tweakcc-fixed` npm name). The rest are real bug fixes against shapes that may have changed again — review each against the current binary before cherry-picking.

If you do this work: `git checkout -b clean-fork upstream/main`, then `git cherry-pick <hash>` per surviving fix. Push to `skrabe/tweakcc-fixed` and update its README to point at upstream as parent.

## When CC releases a new version (the recurring task)

1. Pull upstream prompt JSONs into `tweakcc-fixed/data/prompts/`. They live as `prompts-X.Y.Z.json` and are the source of truth for the pristine prompt text + identifier maps.
2. Run `tweakcc-fixed --apply`. It auto-rebases overrides whose pristine content is unchanged across versions; reports conflicts (with `.diff.html`) for ones where pristine diverged.
3. For conflicts: open the diff HTML, decide whether to keep your override (and update its `ccVersion:` frontmatter) or accept upstream.
4. Run the verification scan below. **This catches a class of bugs that don't show up in conflict reports.**
5. Run any patches (`patchesAppliedIndication`, `userMessageDisplay`, `thinkerFormat`, etc.) and watch for `failed to find …` errors — those mean the minified shape changed and `helpers.ts` regexes need a new method appended.

## Verification rule: every `${VAR}` must exist in the current binary

**The bug class to prevent.** When Anthropic refactors a prompt, they may inline a previously-interpolated variable into literal text. Your override file may still reference the old `${VAR}`, but the variable no longer exists in the pristine `identifierMap`. tweakcc emits the override into the binary's template literal, the JS engine tries to interpolate `${VAR}`, fails with `ReferenceError: VAR is not defined`, and CC crashes on launch.

**The rule.** For each `.md` in `system-prompts/`, every unescaped `${IDENTIFIER}` reference in the body MUST appear as a value in the corresponding pristine prompt's `identifierMap` for the current `ccVersion`. The frontmatter `variables:` list is documentation; the actual contract is with the pristine JSON.

This generalizes — same shape, different variable, different prompt. Run the scan after every CC version bump and after every override edit.

**The check.** Cross-reference each `.md` body's interpolations against the pristine pieces JSON:

```python
# scan_orphan_variables.py
import json, os, re
PROMPTS = '/Users/<you>/dev/tweakcc-fixed/data/prompts/prompts-<latest>.json'
USER_DIR = os.path.expanduser('~/.tweakcc/lobotomized-claude-code/system-prompts')

with open(PROMPTS) as f:
    by_id = {p['id']: p for p in json.load(f)['prompts']}

problems = []
for fname in sorted(os.listdir(USER_DIR)):
    if not fname.endswith('.md'): continue
    pid = fname[:-3]
    if pid not in by_id: continue
    body = re.sub(r'^<!--.*?-->\s*\n?', '', open(os.path.join(USER_DIR, fname)).read(), flags=re.DOTALL)
    # Only flag UN-escaped ${VAR} — backslash before $ means literal in the binary's template literal
    refs = {m.group(2) for m in re.finditer(r'(\\?)\$\{([A-Z_][A-Z0-9_]*)', body) if m.group(1) != '\\'}
    valid = set(by_id[pid].get('identifierMap', {}).values())
    orphan = refs - valid
    if orphan:
        problems.append((pid, sorted(orphan), sorted(valid)))

for pid, orphan, valid in problems:
    print(f"{pid}.md\n  ORPHAN: {orphan}\n  valid:  {valid}\n")
print(f"\nTotal: {len(problems)} files with broken references")
```

**Resolutions per orphan:**

- The variable was renamed → swap to the new name from `identifierMap`.
- The variable was inlined as literal text → check the pristine pieces to see what literal value the binary now uses, and replace `${OLD_VAR}` with that literal.
- The variable is intentionally a literal placeholder (not a JS interpolation) → escape it: `\${OLD_VAR}`. The escape survives template-literal rendering, leaving `${OLD_VAR}` as literal text in the running prompt.

**Note on `variables:` frontmatter.** Keep it in sync with what's actually used in the body, but treat the pristine JSON's `identifierMap` as the binding contract. A variable in `variables:` that's gone from the pristine `identifierMap` is dead metadata and won't help the user; remove it. A `${VAR}` in the body that's NOT in the pristine `identifierMap` is the bug above.

## Other recurring failure modes (generalized)

These are the failure modes I've seen recur, with the diagnostic each time:

- **`patch: findBoxComponent: failed to find Box component`** — Ink's Box wrapper signature changed. `tweakcc-fixed/src/patches/helpers.ts → findBoxComponent` has multiple match methods; add a new one for the current version's shape. Same pattern for `findTextComponent`, `getReactVar`, etc.
- **`patch: thinker format: failed to find approxAreaMatch`** — the destructured-prop signature anchoring this patch changed. Pattern is in `src/patches/thinkerFormat.ts`; add a new anchor based on the current destructure (e.g. find the unique trio of remaining props).
- **`patch: <name>: failed to find <pattern>`** — generic: a regex anchored on minified-but-stable identifiers found a new shape. Open the patch source, copy the failing regex, run it against an extracted `cli.js`, narrow until you find the modern shape, add a new alternative.
- **Prompt apply succeeds but CC crashes on launch** — almost always the orphan-variable bug. Run the scan above.
- **`Could not find system prompt "X" in cli.js`** — your override's content diverges so far from the pristine that the regex (built from the pristine pieces, not your override) doesn't match. Either the override is bound to an older `ccVersion` than the current binary's pristine version, or there's an upstream content change you haven't accepted. Update `ccVersion:` to match, then re-apply.
- **`Auto-escaped unescaped backticks in "X"`** — informational, not an error. tweakcc handles backticks-in-template-literals automatically.
- **dist is stale** — `tweakcc-fixed --apply` reads from `dist/`, not `src/`. Always `pnpm build` after editing `src/`.

## Extracting `cli.js` from a native CC binary

Useful for inspecting current shapes without running tweakcc end-to-end:

```javascript
// extract.mjs
import { extractClaudeJsFromNativeInstallation } from '/Users/<you>/dev/tweakcc-fixed/dist/nativeInstallation-*.mjs';
import fs from 'node:fs';
const r = await extractClaudeJsFromNativeInstallation('/Users/<you>/.nvm/versions/node/<v>/lib/node_modules/@anthropic-ai/claude-code/node_modules/@anthropic-ai/claude-code-darwin-arm64/claude');
fs.writeFileSync('/tmp/cli.js', r.data);
```

Then grep `/tmp/cli.js` directly for the shapes you need to match.

## Day-to-day commands

```bash
# Apply all overrides to the installed CC binary (use the LOCAL build to pick up
# any unpublished tweakcc-fixed patches like max-effort default).
node ~/dev/tweakcc-fixed/dist/index.mjs --apply

# Restore CC to its pristine state (preserves config.json).
node ~/dev/tweakcc-fixed/dist/index.mjs --restore

# Open the tweakcc UI to toggle the non-prompt patches (themes, max-effort, etc.).
node ~/dev/tweakcc-fixed/dist/index.mjs

# After editing tweakcc-fixed source, rebuild before --apply picks up changes.
cd ~/dev/tweakcc-fixed && pnpm build

# Sync overrides to GitHub.
cd ~/.tweakcc/lobotomized-claude-code && git add -A && git commit -m "..." && git push origin main
```

## Don't use `npx tweakcc-fixed@latest --apply`

That pulls the published `1.0.4` from npm, which doesn't have the local patches the user has added (e.g. max-effort default, any future patch additions). Always use the local build via `node ~/dev/tweakcc-fixed/dist/index.mjs`. The npm package is reference-quality, not the user's actual workflow.

## Things to surface (not assume)

- If you're about to delete or replace a prompt the user spent time editing, check `git log -- system-prompts/<file>.md` first and surface what you'd lose.
- If you're about to commit changes the user didn't ask for, ask. The user is opinionated about what goes into their fork.
- If a CC release breaks more patches than you can fix in one pass, list what fails and what would be needed; let the user decide priority.
- If the user mentions a different prompt-engineering pattern (e.g. a new skill they like, a fix from someone's blog post), incorporate it — they're tracking the ecosystem actively.

## Personality of the work

The user values:
- Direct, specific writing — no "best practices" hand-waving, no AI-voice softeners
- Real numbers over claims — measure tokens before claiming a cut
- Surgical edits — don't refactor adjacent stuff that wasn't asked
- Tables and bullets when they earn their keep, not as default
