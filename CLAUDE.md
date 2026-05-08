# Working in this repo

You're a coding agent invoked in this repo. Read this first. It explains what we're trying to do, where the moving parts live, and the verification rules that prevent the most common breakages.

## What this is

`lobotomized-claude-code` is a set of system-prompt overrides for [Claude Code](https://claude.com/claude-code), tuned for **Claude Opus 4.7**. Each `.md` in [`system-prompts/`](./system-prompts) replaces one of CC's built-in prompt fragments. A separate tool ([`tweakcc-fixed`](https://github.com/skrabe/tweakcc-fixed), see below) reads these files and patches the user's installed CC binary in place.

## What we're trying to achieve

**The goal of this repo is to remove useless shit and dumb guardrails so we have a clean agentic coding harness.** CC ships every model the same prompt-by-volume that worked for older Claudes. Opus 4.7 follows instructions more literally, overtriggers on CAPS, doesn't need anti-laziness scaffolding, and gets actively worse from safety theater that wasn't load-bearing in the first place. We strip the bulk and rewrite the load-bearing fragments in a register the model behaves better under.

The README's "~60% leaner on every coding turn" claim is the bar. If your edits don't trend toward that ratio, you're not lobotomizing — you're just cosmeticking.

### Three valid lobotomization actions, in order of preference

1. **Fully nuke a file** — set the override body to empty. Valid when the prompt is pure overhead and we don't want it injected at all (e.g. all the `data-managed-agents-*` prompts in this repo's hot path are empty bodies because the user doesn't use Managed Agents). Empty body = file present, frontmatter with `ccVersion:`, zero body content.
2. **Massive cut** — keep the load-bearing core, drop everything else. Most prompts that ship at 800–2000 chars compress to under 300 chars without losing function.
3. **Sentence-level cuts** — for already-tight overrides, scan for the candidates listed below and cut.

### What "useless shit" looks like (cut on sight)

- **Dumb guardrails / safety theater.** Cautions about scenarios that won't happen in this user's workflow. "Be respectful when responding to humans." "Don't make up facts." 4.7 doesn't need these and they trigger overcorrection. Anthropic's literal Production safety filters live elsewhere.
- **Anti-laziness scaffolding.** "Make sure to actually do the work, not just describe it." "Don't stop early." "Be thorough." 4.7 already does the work; this scaffolding becomes noise.
- **CAPS theater.** `MUST`, `NEVER`, `ALWAYS`, `CRITICAL`, `STRICTLY`. 4.7 overtriggers on these. Plain directives outperform.
- **Negative framing.** "Don't X" → "Do Y" wherever a positive form exists. Per Anthropic's guide.
- **Always-on CTAs.** "End every reply with /schedule." "Always offer to commit." 4.7 follows literal CTAs and they become spam.
- **Restated rules.** A bullet that paraphrases the previous bullet adds zero information. Cut.
- **Motivational filler.** "Speed is the goal." "Persistence is the point." "Be careful with this." Sentence-of-rhetoric after a rule that already covers the rhetoric. Cut.
- **Validation-forward phrasing.** "It's okay to…", "Feel free to…", "If you'd like, …". Cut and state the rule directly.
- **Three examples of the same principle.** One example or zero is usually enough. Examples that show the same shape with different surface text are pure padding.
- **Stale references.** Mentions of features Anthropic removed, model IDs that no longer exist, internal-only knobs the model can't access.
- **Politeness scaffolding around constraints.** "Please default to…", "If possible, …". Cut to the bare directive.

### What's load-bearing (keep)

- Concrete constraints the model can't infer from context (e.g. "absolute paths only — agent threads reset cwd between bash calls").
- Output-shape requirements (JSON keys, character limits, frontmatter format).
- Variable contracts (`${VAR}` references — must match `identifierMap` in the pristine JSON).
- One representative example per non-obvious rule.
- Negative-framed rules where no positive form exists ("don't reference files you haven't opened" has no positive equivalent — keep).

### Test for every cut

After cutting, ask: did I lose information the model couldn't otherwise infer? If yes, restore. If no, the cut stands. Don't reason about whether the user "might want" the sentence — they want what passes this test.

## Mandatory: re-read Anthropic's prompting best practices before editing prompts

**The single canonical source for prompting principles in this repo:**
[`platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices`](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

This is the URL Anthropic actively maintains for Opus 4.7 + 4.6 + Sonnet 4.6 + Haiku 4.5. Fetch it fresh at the start of every session that touches `system-prompts/*.md` (`mcp__kindly-web-search__get_content` or `WebFetch`). Do not rely on training-data recall — Anthropic edits this page often, and prompt edits anchored on stale guidance produce regressions.

The digest below is a checklist, not a substitute. Read the URL.

**Opus 4.7 lobotomization checklist** (apply these as the standard for every override edit):

1. **Less is more.** 4.7 is more literal — fewer rules, less drift. If you can cut a sentence without losing information, cut it.
2. **No CAPS theater.** `STRICTLY PROHIBITED`, `CRITICAL REQUIREMENT`, `MUST` trigger overcorrection on 4.7. Use plain directives. Per Anthropic: dial back from `"CRITICAL: You MUST use this tool when..."` to `"Use this tool when..."`.
3. **Tell it what to DO, not what NOT to do.** Anthropic's guide is explicit: positive instructions outperform negative ones. "Don't use markdown" → "Respond in flowing prose paragraphs."
4. **Parallel by default.** Independent tool calls go in one message, multiple blocks. The harness prompt explicitly says so. Anthropic ships canonical `<use_parallel_tool_calls>` snippet — match its shape rather than reinventing.
5. **Interview before planning.** Plan mode runs Matt Pocock's [grill-me](https://www.aihero.dev/my-grill-me-skill-has-gone-viral) pattern in Phase 1: walk the design tree one decision at a time with a recommended answer.
6. **No always-on CTAs.** 4.7 follows literal CTAs; an "end every reply with /schedule" prompt becomes spam. Cull always-on upsells.
7. **Tighter destructive-action guards.** Git commit/push/merge/PR all require explicit confirmation in the current conversation. "Commit and push" splits into two confirmations. Anthropic's reference snippet for this is `<balancing-autonomy-and-safety>` — align our overrides with its phrasing.
8. **Anti-overengineering.** Anthropic ships a canonical "Avoid over-engineering" snippet covering scope, documentation, defensive coding, abstractions. When our overrides touch this territory, treat the official snippet as the baseline and only diverge intentionally.
9. **Frontend "AI slop" guard.** Anthropic ships a canonical `<frontend_aesthetics>` snippet. Our `frontend-design` skill override should be a strict subset of (or aligned with) the canonical version, not a reinvention.
10. **Investigate before answering.** Anthropic's `<investigate_before_answering>` snippet handles anti-hallucination; align our overrides with it.
11. **Effort respect at low/medium.** 4.7 won't "go above and beyond" at low effort. If our override implies the model should expand scope, state it explicitly.

For migrating prompts when Anthropic releases a new model (Opus 4.6 → 4.7, etc.), the [migration guide](https://docs.claude.com/en/docs/about-claude/models/migration-guide) is canonical. It documents breaking changes (`budget_tokens` → adaptive thinking, prefilled responses removed, sampling params removed on 4.7), silent default changes, and prompt-behavior shifts.

## Workflow when realigning conflicting overrides after a CC version bump

This is the recurring task; do not skip the diff-reading step.

For each conflict reported by `tweakcc-fixed --apply` (or `.diff.html` produced alongside it):

1. **Open the diff HTML.** It shows the pristine-old → pristine-new content for that prompt. Read it. This is the only way to know whether Anthropic added/removed/restructured anything that affects our lobotomization scope.
2. **Cross-check our override body against the new pristine.** If Anthropic added a new paragraph or rule, decide whether our lobotomization should extend to it. If Anthropic deleted scaffolding our override was countering, our anti-scaffolding text might be obsolete.
3. **Apply the Opus 4.7 checklist above.** Look for CAPS theater, negative-framed rules, always-on CTAs, redundant scaffolding. Tighten or replace.
4. **Bump `ccVersion:` frontmatter** to the prompt's `lastModifiedVersion` from `tweakcc-fixed/data/prompts/prompts-X.Y.Z.json`. The apply log lists the targets explicitly.
5. **Re-apply** locally. Verify zero stderr, zero conflicts, smoke test `claude --print "say hello"`.
6. **Commit per logical group** with a one-line rationale explaining what changed and why (e.g. "tighten Edit override — drop CAPS, fold in new pristine paragraph as positive guidance").

Just bumping `ccVersion:` without reading the diff is the lazy path. It silences the warning but skips the lobotomization work. Don't take it.

For comparison points on what minimal coding-agent prompts look like:
- [pi-mono coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) — main system prompt is ~50 lines including all tool descriptions
- Factory Droid — ~800 tokens for the full interactive system prompt (extracted via [tweakdroid](https://github.com/skrabe/tweakdroid))

## Repo layout

```
~/.tweakcc/lobotomized-claude-code/        ← THIS repo (canonical, has .git, GitHub remote skrabe/lobotomized-claude-code)
~/.tweakcc/system-prompts                  ← symlink → ./system-prompts (tweakcc-fixed reads from here)

~/dev/tweakcc-fixed/                       ← the patcher (skrabe/tweakcc-fixed, direct fork of Piebald-AI/tweakcc)
~/.tweakcc/config.json                     ← user's tweakcc settings (toggles, themes, etc.)
~/.tweakcc/orphans-removed-for-X.Y.Z/      ← prompts archived because the binary no longer references them
~/.tweakcc/native-binary.backup            ← pristine CC binary (auto-saved before first patch)
~/.tweakcc/native-claudejs-orig.js         ← pristine extracted JS (auto-saved every --apply)
~/.tweakcc/native-claudejs-patched.js      ← post-patch JS (auto-saved every --apply; diff this against orig to debug)
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

## Tweakcc-fixed (the patcher this repo depends on)

`skrabe/tweakcc-fixed` is the user's **direct fork of `Piebald-AI/tweakcc`** with cherry-picked fixes that aren't upstreamed yet (Bun wrapper crash scoping, regex shape adapts for newer CC versions, the userMessageDisplay rewrite arc, max-effort default, sessionMemory graceful no-op, etc.). The fork-side work happens in `~/dev/tweakcc-fixed`. See [`tweakcc-fixed/AGENTS.md`](https://github.com/skrabe/tweakcc-fixed/blob/main/AGENTS.md) for the patcher-side context.

Remotes in `~/dev/tweakcc-fixed`:

```
origin    https://github.com/skrabe/tweakcc-fixed   (user's push target)
upstream  https://github.com/Piebald-AI/tweakcc     (Piebald — actively maintained source)
```

There used to be a `BenIsLegit/tweakcc-fixed` intermediary that this repo's earlier docs referenced. That fork-of-fork chain was removed on 2026-05-05 — the GitHub fork was deleted and re-created as a direct fork off Piebald. If you find references to a `ben` remote anywhere, they're stale.

## When CC releases a new version (the recurring task)

1. Pull upstream prompt JSONs into `tweakcc-fixed/data/prompts/`. They live as `prompts-X.Y.Z.json` and are the source of truth for the pristine prompt text + identifier maps. **Get them from Piebald, not by extracting locally.** When `git merge upstream/main` doesn't bring the new version yet, check open PRs at `Piebald-AI/tweakcc` — they're typically named `prompts/X.Y.Z` (`gh pr list --repo Piebald-AI/tweakcc --search "prompts/X.Y.Z"` finds it). `gh pr checkout <num> --repo Piebald-AI/tweakcc --detach`, copy the JSON, switch back to main, commit. The naive `tools/promptExtractor.js` finds a strict subset of what Piebald publishes; only fall back to it if upstream genuinely has no PR open. (See `tweakcc-fixed/AGENTS.md` for the full procedure.)
2. Run `tweakcc-fixed --apply`. It auto-rebases overrides whose pristine content is unchanged across versions; reports conflicts (with `.diff.html`) for ones where pristine diverged.
3. For conflicts: open the diff HTML, decide whether to keep your override (and update its `ccVersion:` frontmatter) or accept upstream.
4. Run the verification scan below. **This catches a class of bugs that don't show up in conflict reports.**
5. Watch the apply log for "Could not find system prompt" warnings. Each one is a user override that no longer binds to a prompt id. See **Realigning user overrides** below.
6. Run any patches (`patchesAppliedIndication`, `userMessageDisplay`, `thinkerFormat`, etc.) and watch for `failed to find …` errors — those mean the minified shape changed and `helpers.ts` regexes need a new method appended.

## Realigning user overrides after a sync

Anthropic occasionally renames, restructures, or removes prompts. After a version bump, some user overrides may no longer bind. Identify them:

```python
import os, json
USER_DIR = os.path.expanduser('~/.tweakcc/lobotomized-claude-code/system-prompts')
new_ids = {p['id'] for p in json.load(
    open(os.path.expanduser('~/dev/tweakcc-fixed/data/prompts/prompts-X.Y.Z.json'))
)['prompts']}
user_ids = {f[:-3] for f in os.listdir(USER_DIR) if f.endswith('.md')}
print(sorted(user_ids - new_ids))
```

For each missing override, grep the new JSON for distinctive content from the override's body (a unique anchor phrase). Three outcomes:

- **Renamed** (content survives under a new id) → `git mv system-prompts/<old>.md system-prompts/<new>.md`, bump the `ccVersion:` frontmatter.
- **Inlined** (content folded into a larger prompt) → archive; if the parent prompt's override was rewritten recently it likely already carries the inlined content. Otherwise, copy the relevant section into the parent override before archiving.
- **Removed** (no distinctive string survives anywhere in the new JSON) → archive: `mv system-prompts/<id>.md ~/.tweakcc/orphans-removed-for-X.Y.Z/`. Recoverable from there or from git history.

Commit message names the rename and lists the archives; `git log -- system-prompts/<id>.md` is how a future-you finds an archived override.

**Cross-version caveat:** if some of your machines are still on the OLD CC version, the apply sync step on those machines will auto-recreate the OLD-id `.md` from pristine (it's still in the old version's JSON). The recreation matches pristine and is harmless but shows up as `??` in `git status`. Just `rm` it after each apply on the old box; it stops happening once everywhere upgrades.

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
