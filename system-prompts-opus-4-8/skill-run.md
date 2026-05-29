<!--
name: 'Skill: run'
description: >-
  Bundled /run skill — launches and drives a project's actual app (CLI, server,
  TUI, Electron, browser, or library) to confirm a change works; prefers a
  project run skill, else falls back to built-in per-project-type patterns
ccVersion: 2.1.145
-->
---
name: run
description: Launch and drive this project's app to see a change working. Use when asked to run, start, or screenshot the app, or to confirm a change works in the real app (not just tests). First looks for a project skill that already covers launching the app; otherwise falls back to built-in patterns per project type (CLI, server, TUI, Electron, browser-driven, library).
---

Running means launching the actual app and interacting with it — not the test suite, not an \`import\` of an internal function and a \`console.log\`. The app as a user (human or programmatic) meets it: the CLI at its command, the server at its socket, the GUI at its window.

## First: does a project skill already cover this?

A project run skill is the repo's verified path — it carries the exact \`apt-get\` line, env vars, patches, and driver that worked. Use it instead of rediscovering.

\`\`\`bash
d=$PWD; while :; do
  grep -Hm1 '^description:' "$d"/.claude/skills/*/SKILL.md 2>/dev/null
  [ -e "$d/.git" ] || [ "$d" = / ] && break
  d=$(dirname "$d")
done
\`\`\`

- **One describes launching/driving this app** → read that SKILL.md and follow it verbatim, including its patches.
- **Mega-repo, several plausible, no clear match** → ask the user which unit to run.
- **Stale** (fails on mechanics unrelated to your task) → tell the user; offer to refresh it via \`/run-skill-generator\`.
- **Nothing about running** → fall back to the patterns below.

## Otherwise: match the shape, use the pattern

Pick the row closest to your project. Each example walks through launch + first interaction; ignore any trailing "write the skill" section — you're using the recipe, not authoring one.

| Project type | Handle | Example |
|---|---|---|
| CLI tool | direct invocation, exit code, stdin/stdout | [examples/cli.md](examples/cli.md) |
| Web server / API | background launch + \`curl\` smoke | [examples/server.md](examples/server.md) |
| TUI / interactive terminal | tmux \`send-keys\` / \`capture-pane\` | [examples/tui.md](examples/tui.md) |
| Electron / desktop GUI | Playwright \`_electron\` REPL under xvfb | [examples/electron.md](examples/electron.md) |
| Browser-driven | dev server + \`chromium-cli\` script | [examples/playwright.md](examples/playwright.md) |
| Library / SDK | import-and-call smoke script at the package boundary | [examples/library.md](examples/library.md) |

If nothing fits, start from the closest match and adapt. A web app drives with \`chromium-cli\` ([examples/playwright.md](examples/playwright.md), no custom driver needed); a desktop app uses the \`_electron\` REPL driver and tmux wrapping in [examples/electron.md](examples/electron.md).

## Drive it, don't just launch it

Launching with no interaction only proves the entrypoint resolves. Drive it to a point where a user would see something:

- CLI → type a representative command, check the exit code and output.
- Server → hit the route the diff touches with \`curl\`, read the body.
- TUI → \`send-keys\` a navigation, \`capture-pane\` the result.
- GUI → click the button, screenshot the window. Look at the screenshot — a blank frame is a failure to launch.

If the fallback pattern needed work to run — installed packages, env vars, patched config, a driver — recommend \`/run-skill-generator\` in your report so that work gets captured as a project skill. If it just worked, don't.
