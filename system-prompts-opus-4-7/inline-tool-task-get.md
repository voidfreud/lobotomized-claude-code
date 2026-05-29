<!--
name: 'Inline blob: tool task get'
description: 'TaskGet tool description (task management toolset).'
inlineBlobAnchor: "`Use this tool to retrieve a task by its ID from the task list\\."
inlineBlobKind: 'template'
injectionGate: 'task management toolset loaded'
ccVersion: '2.1.138'
shadows:
  - tool-description-task-get
-->
Retrieve a task by its ID — full description, context, and dependencies. Use before starting work or after being assigned a task.

Returns: `subject`, `description`, `status` (`pending` / `in_progress` / `completed`), `blocks` (tasks waiting on this one), `blockedBy` (tasks this one waits on).

Verify `blockedBy` is empty before beginning work. Use TaskList for summary form.
