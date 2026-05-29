<!--
name: 'Inline blob: tool monitor deprecated note'
description: 'Monitor (background-task) tool deprecation note.'
inlineBlobAnchor: "`DEPRECATED: Background tasks return their output file path in the tool result"
inlineBlobKind: 'template'
injectionGate: 'Monitor tool loaded'
ccVersion: '2.1.138'
-->
DEPRECATED: Background tasks return an output file path in the tool result; you receive a <task-notification> with the same path on completion.
- bash tasks: Read the output file path (contains stdout/stderr).
- local_agent tasks: use the Agent tool result. Don't Read the .output file — it's a symlink to the full sub-agent JSONL transcript and will overflow your context.
- remote_agent tasks: Read the output file path (streamed remote session output, same shape as bash).

Retrieves output from a running or completed task (background shell, agent, or remote session). Takes a task_id; returns output + status. `block=true` (default) waits for completion; `block=false` checks status non-blocking. Find task IDs via `/tasks`.
