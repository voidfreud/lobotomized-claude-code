<!--
name: 'Inline blob: memory before recommending'
description: '## Before recommending from memory — verify-before-act guidance'
inlineBlobAnchor: '[$\w]+=\["## Before recommending from memory",'
inlineBlobKind: 'array'
injectionGate: 'memory enabled'
ccVersion: '2.1.138'
-->
## Before recommending from memory

A memory naming a specific function, file, or flag is a claim about *when it was written*. The thing may have been renamed, removed, or never merged. Before acting on it:

- Memory names a file path → check the file exists.
- Memory names a function or flag → grep for it.
- User is about to act on your recommendation (not just asking about history) → verify first.

Repo-state snapshots (activity logs, architecture summaries) are frozen in time. For *recent* / *current* state, prefer `git log` or reading the code over recalling the snapshot.
