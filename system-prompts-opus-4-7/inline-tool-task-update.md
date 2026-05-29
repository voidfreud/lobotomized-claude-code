<!--
name: 'Inline blob: tool task update'
description: 'TaskUpdate tool description (task management toolset).'
inlineBlobAnchor: "`Use this tool to update a task in the task list\\."
inlineBlobKind: 'template'
injectionGate: 'task management toolset loaded'
ccVersion: '2.1.138'
shadows:
  - tool-description-task-update
-->
Update a task in the task list.

Mark assigned tasks resolved when you finish them, then call TaskList for the next. Only mark `completed` when the work is fully done — keep `in_progress` if tests fail, implementation is partial, errors are unresolved, or files/deps are missing. When blocked, create a new task describing what needs resolving.

Use `deleted` to permanently remove a task created in error or no longer relevant.

## Fields

- `status` — see workflow below
- `subject` — imperative title (e.g., "Run tests")
- `description` — task description
- `activeForm` — present-continuous form shown in spinner during `in_progress` (e.g., "Running tests")
- `owner` — agent name
- `metadata` — merge keys; set a key to `null` to delete
- `addBlocks` / `addBlockedBy` — dependencies

## Status

`pending` → `in_progress` → `completed`. Use `deleted` to remove.

Read a task's latest state via `TaskGet` before updating.

## Examples

```json
{"taskId": "1", "status": "in_progress"}
{"taskId": "1", "status": "completed"}
{"taskId": "1", "status": "deleted"}
{"taskId": "1", "owner": "my-name"}
{"taskId": "2", "addBlockedBy": ["1"]}
```
