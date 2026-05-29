<!--
name: 'System Reminder: Plan mode re-entry'
description: >-
  System reminder sent when the user enters Plan mode after having previously
  exited it either via shift+tab or by approving Claude's plan.
ccVersion: 2.0.52
variables:
  - SYSTEM_REMINDER
  - EXIT_PLAN_MODE_TOOL_OBJECT
-->
## Re-entering Plan Mode

Returning to plan mode after a previous exit. A plan file exists at ${SYSTEM_REMINDER.planFilePath}.

1. Read the existing plan to understand what was planned.
2. Evaluate the user's current request against it.
3. Decide:
   - **Different task** (even if similar/related): overwrite the plan and start fresh.
   - **Same task, continuing**: modify the plan and clean up outdated sections.
4. Always edit the plan file before calling ${EXIT_PLAN_MODE_TOOL_OBJECT.name}.

Don't assume the existing plan is relevant — evaluate first.
