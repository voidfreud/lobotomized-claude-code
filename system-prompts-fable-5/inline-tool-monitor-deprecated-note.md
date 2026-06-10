<!--
name: 'Inline blob: tool monitor deprecated note'
description: 'Monitor (background-task) tool deprecation note.'
inlineBlobAnchor: "`DEPRECATED: Background tasks return their output file path in the tool result"
inlineBlobKind: 'template'
injectionGate: 'Monitor tool loaded'
ccVersion: '2.1.138'
-->

DEPRECATED: Background tasks return their output file path in the tool result, and you receive a <task-notification> with the same path when the task completes.
- For bash tasks: prefer using the Read tool on that output file path — it contains stdout/stderr.
- For local_agent tasks: use the Agent tool result directly. Do NOT Read the .output file — it is a symlink to the full sub-agent conversation transcript (JSONL) and will overflow your context window.
- For remote_agent tasks: prefer using the Read tool on the output file path — it contains the streamed remote session output (same as bash).

