<!--
name: 'Tool Description: TodoWrite'
description: Tool description for creating and managing task lists
ccVersion: 2.1.84
variables:
  - EDIT_TOOL_NAME
-->
Create and manage a structured task list for the current session. Use for 3+ step tasks, multi-task user requests, or capturing new requirements as you receive them. Skip for single trivial tasks.

Lifecycle: mark `in_progress` BEFORE starting work — only one task `in_progress` at a time. Mark `completed` immediately after finishing (don't batch). Don't mark `completed` if tests fail, implementation is partial, or required files are missing — keep `in_progress` and create a new task for the blocker.

Each task has two forms: `content` (imperative, e.g. "Run tests") and `activeForm` (present continuous, shown in the spinner during `in_progress`, e.g. "Running tests"). Status values: `pending`, `in_progress`, `completed`. Remove tasks that become irrelevant.
