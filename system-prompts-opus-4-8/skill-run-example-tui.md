<!--
name: 'Skill: run example — TUI / interactive terminal app'
description: >-
  Bundled example doc (examples/tui.md) for the run skill: driving an
  interactive terminal app by wrapping it in tmux send-keys / capture-pane
ccVersion: 2.1.145
-->
# Example: TUI / interactive terminal app

Interactive terminal apps (text editors, REPLs, curses UIs) take over the terminal, so the agent can't drive them directly. Wrap them in \`tmux\` to send input, capture output, and screenshot.

## The tmux pattern

1. Start the TUI in a detached tmux session.
2. Send keystrokes with \`tmux send-keys\`.
3. Read the screen with \`tmux capture-pane\`.
4. Clean up with \`tmux kill-session\`.

Present this as the primary way to drive the app in \`SKILL.md\`. A small \`driver.sh\` wrapping launch+attach can live in the skill directory, but for most TUIs the raw tmux commands suffice.

## Example snippet

> ## Run (interactive, for agents)
>
> Start the TUI inside tmux:
>
> \`\`\`bash
> tmux new-session -d -s app -x 120 -y 40 './myapp'
> \`\`\`
>
> Poll until the ready marker appears (returns the instant the app is up, fails loudly if it isn't):
>
> \`\`\`bash
> timeout 10 bash -c 'until tmux capture-pane -t app -p | grep -q "Ready"; do sleep 0.2; done'
> tmux capture-pane -t app -p
> \`\`\`
>
> Send input (navigate to Settings and toggle an option):
>
> \`\`\`bash
> tmux send-keys -t app 's'
> timeout 5 bash -c 'until tmux capture-pane -t app -p | grep -q "Settings"; do sleep 0.2; done'
> tmux send-keys -t app 'Down' 'Down' 'Space'  # navigate + toggle
> timeout 5 bash -c 'until tmux capture-pane -t app -p | grep -qF "[x]"; do sleep 0.2; done'
> tmux capture-pane -t app -p
> \`\`\`
>
> If you write more than a couple of these poll lines, pull them into a \`wait_for()\` helper in a \`driver.sh\` next to the skill.
>
> Quit:
>
> \`\`\`bash
> tmux send-keys -t app 'q'
> tmux kill-session -t app 2>/dev/null || true
> \`\`\`
>
> ### Key reference
>
> | Key | Action |
> |---|---|
> | \`j\` / \`k\` or \`Down\` / \`Up\` | Navigate list |
> | \`Enter\` | Select |
> | \`s\` | Settings |
> | \`q\` | Quit |

## Details worth documenting

- **Terminal size.** Some TUIs break or hide content at small widths. Specify a known-good size in the \`tmux new-session -x -y\` args.
- **Ready marker.** Poll for it (\`until tmux capture-pane | grep -q X\`) rather than a fixed \`sleep N\`. Say what string means ready.
- **Keybinding reference.** A table of the main keys — this is the TUI's "API."
- **Exit cleanly.** Show the quit keystroke *and* \`tmux kill-session\` as a fallback.
- **Color/unicode quirks.** If \`capture-pane\` output is hard to read, note flags that help (\`-e\` for escape sequences, \`-J\` to join wrapped lines).

## Also document the direct invocation

For a human running interactively, tmux is overkill. Include the one-liner:

> ## Run (direct, for humans)
>
> \`\`\`bash
> ./myapp
> \`\`\`
>
> Press \`q\` to quit.
