<!--
name: 'Skill: run-<unit-name> (template)'
description: >-
  Template skeleton for a per-project run-<unit-name> skill, bundled as
  template.md inside run-skill-generator — prerequisites, setup, build,
  agent/human run paths, test, and gotchas sections
ccVersion: 2.1.145
-->
---
name: run-<unit-name>
description: Build, run, and drive <unit-name>. Use when asked to start <unit-name>, run its tests, build it, take a screenshot of its UI, or interact with the running app.
---

<One-sentence description: what this is and how an agent drives it.
Name the handle — "drive it via
\`.claude/skills/run-<unit-name>/driver.mjs\` under xvfb" for a desktop
app, or "start the dev server then drive it via \`chromium-cli\`" for a
web app.>

<If the unit isn't at repo root:>
All paths below are relative to \`<unit-dir>/\`.

## Prerequisites

<The exact \`apt-get install\` line you ran — the one that actually
worked, not a generic list. Target Ubuntu.>

\`\`\`bash
sudo apt-get update
sudo apt-get install -y <packages-you-actually-installed>
\`\`\`

<Runtime versions if they matter:>

\`\`\`bash
# Example: Node 20 via nvm, Python 3.12 via uv, etc.
\`\`\`

## Setup

<One-time setup after clone: install deps, configure, apply any
patches (feature-gate overrides, config stubs) with the exact command.>

\`\`\`bash
<commands>
\`\`\`

<Env vars — required vs optional, with defaults:>

\`\`\`bash
export FOO_API_KEY=...   # required — get from <where>
export BAR_MODE=dev      # optional — default is prod
\`\`\`

## Build

<Skip if no separate build step. Otherwise the exact command:>

\`\`\`bash
<command>
\`\`\`

## Run (agent path)

<The section a future agent uses. If you built a driver/REPL/smoke
script, document how to launch it and what it does. If a \`curl\`
one-liner suffices, that goes here.>

\`\`\`bash
<launch-the-driver-or-smoke-script>
\`\`\`

<For REPL-style drivers, show the tmux wrapping. Poll for a ready
marker between send-keys and capture-pane — faster than a fixed sleep,
and fails loudly instead of capturing a half-rendered screen:>

\`\`\`bash
tmux new-session -d -s app -x 200 -y 50
tmux send-keys -t app '<launch command>' Enter
timeout 30 bash -c 'until tmux capture-pane -t app -p | grep -q "<ready-marker>"; do sleep 0.2; done'
tmux send-keys -t app '<first driver command>' Enter
tmux capture-pane -t app -p
\`\`\`

<Where artifacts land — absolute paths:>

Screenshots → \`/tmp/shots/\`. Logs → \`/tmp/<app>.log\`.

<If the driver has commands, a table:>

| command | what it does |
|---|---|
| \`<cmd>\` | <description> |

## Run (human path)

<If meaningfully different from the agent path. Brief.>

\`\`\`bash
<command>   # → <what happens>. <how to stop>.
\`\`\`

## Test

\`\`\`bash
<command>
\`\`\`

<Expected result — "N suites pass", or specific known-flaky tests.>

---

<Optional sections below — include only with content you actually hit,
not generic advice.>

## Gotchas

<Non-obvious traps: looks like it should work but doesn't, with the
workaround. Generic → delete this section.>

- **<specific thing>** — <why it breaks> → <what to do instead>

## Troubleshooting

<Symptom → fix. Only errors you actually encountered.>

- **<exact error message or symptom>**: <cause>. <fix>.

<---

NOTES:
- Replace <unit-name> in \`name:\` (becomes /run-<unit-name>, must match
  the directory name) and \`description:\`. Keep the description verbs —
  start, run, build, test, screenshot — they're what an asking agent types.
- A driver script lives in this directory by default; reference it from
  Run. A web app usually has no driver — the \`chromium-cli\` heredoc is
  the harness. If a driver grows into shared launch helpers or a real
  e2e harness, move it to scripts/ or e2e/ and update the paths here.

Delete everything from \`---\` above onwards before committing. --->
