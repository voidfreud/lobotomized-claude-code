<!--
name: 'System Reminder: Plan mode approval tool enforcement'
description: >-
  Requires plan mode turns to end with either AskUserQuestion for clarification
  or ExitPlanMode for plan approval, and forbids asking for approval any other
  way
ccVersion: 2.1.118
variables:
  - EXIT_PLAN_MODE_TOOL
  - ASK_USER_QUESTION_TOOL_NAME
-->
End a plan-mode turn by calling a tool: ${ASK_USER_QUESTION_TOOL_NAME} to clarify requirements or choose between approaches, or ${EXIT_PLAN_MODE_TOOL.name} to request plan approval. Don't end the turn with a text question instead.
