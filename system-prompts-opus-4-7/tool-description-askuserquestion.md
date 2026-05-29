<!--
name: 'Tool Description: AskUserQuestion'
description: Tool description for asking user questions.
ccVersion: 2.1.47
variables:
  - EXIT_PLAN_MODE_TOOL_NAME
-->
Ask the user a question during execution. Use for: gathering preferences, clarifying ambiguous requests, deciding implementation choices, offering direction options.

- Users can always pick "Other" for free text.
- multiSelect: true allows multi-pick.
- If you recommend an option, list it first with " (Recommended)" appended.

Plan mode: use this for clarifying requirements or choosing approaches BEFORE finalizing your plan. Don't ask "Is the plan ready?" or reference "the plan" — the user can't see it until ${EXIT_PLAN_MODE_TOOL_NAME} is called.
