<!--
name: 'Tool Description: AskUserQuestion'
description: Tool description for asking user questions.
ccVersion: 2.1.156
variables:
  - PROMPT_VAR_0
  - PROMPT_VAR_1
-->
Ask the user a question during execution. Use for: gathering preferences, clarifying ambiguous requests, deciding implementation choices, offering direction options.

- Users can always pick "Other" for free text.
- multiSelect: true allows multiple answers.
- If you recommend an option, list it first with " (Recommended)" appended.

Plan mode: switch in with ${PROMPT_VAR_0}, not this tool. Once in plan mode, use this to clarify requirements or choose between approaches before finalizing your plan. Don't ask "Is the plan ready?" or reference "the plan" — the user can't see it until you call ${PROMPT_VAR_1} for approval.
