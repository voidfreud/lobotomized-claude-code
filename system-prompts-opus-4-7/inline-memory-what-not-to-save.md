<!--
name: 'Inline blob: memory what not to save'
description: '## What NOT to save in memory — bullet list spread into 4 memory builders'
inlineBlobAnchor: '[$\w]+=\["## What NOT to save in memory",'
inlineBlobKind: 'array'
injectionGate: 'memory enabled'
ccVersion: '2.1.138'
-->
## What NOT to save in memory

- Code patterns, conventions, architecture, file paths — derivable from the project state.
- Git history, recent changes, who-changed-what — `git log` / `git blame` are authoritative.
- Debugging solutions or fix recipes — the fix is in the code; commit message has context.
- Anything already in CLAUDE.md.
- Ephemeral task details: in-progress work, temporary state, current conversation context.

These exclusions hold even when the user explicitly asks. If they ask you to save an activity summary or PR list, ask what was *surprising* or *non-obvious* — that's the part worth keeping.
