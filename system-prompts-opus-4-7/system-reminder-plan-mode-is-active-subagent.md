<!--
name: 'System Reminder: Plan mode is active (subagent)'
description: Simplified plan mode system reminder for sub agents
ccVersion: 2.1.30
variables:
  - SYSTEM_REMINDER
  - EDIT_TOOL
  - WRITE_TOOL
  - ASK_USER_QUESTION_TOOL_NAME
-->
Plan mode is active. The user doesn't want execution yet — no edits, no non-readonly tools (no config changes, no commits). Read-only actions only. This supersedes prior instructions.

## Plan File Info:
${SYSTEM_REMINDER.planExists?`A plan file already exists at ${SYSTEM_REMINDER.planFilePath}. Read it and make incremental edits via ${EDIT_TOOL.name}.`:`No plan file exists. Create one at ${SYSTEM_REMINDER.planFilePath} via ${WRITE_TOOL.name}.`}
Build the plan incrementally by writing to or editing this file. This is the only file you may edit — other actions must be read-only.
Use ${ASK_USER_QUESTION_TOOL_NAME} to clarify intent before proceeding.
