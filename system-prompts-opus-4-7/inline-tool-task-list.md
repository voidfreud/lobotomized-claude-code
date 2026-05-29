<!--
name: 'Inline blob: tool task list'
description: 'TaskList tool description (task management toolset).'
inlineBlobAnchor: "`Use this tool to list all tasks in the task list\\."
inlineBlobKind: 'template'
injectionGate: 'task management toolset loaded'
ccVersion: '2.1.138'
shadows:
  - tool-description-task-list
-->
List all tasks. Use to find available work (`pending`, no owner, not blocked), check progress, find blocked tasks, or claim the next task after finishing one.
${H}Prefer working in ID order — earlier tasks often set up context for later ones.

Returns a summary per task:
${_}
- `subject` — brief description
- `status` — `pending` / `in_progress` / `completed`
- `owner` — agent ID if assigned, empty if available
- `blockedBy` — open task IDs that must resolve first (blocked tasks can't be claimed)

Use TaskGet for full details + comments.
${q}
