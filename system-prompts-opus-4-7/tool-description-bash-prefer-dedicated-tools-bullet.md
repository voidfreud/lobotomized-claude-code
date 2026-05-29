<!--
name: 'Tool Description: Bash (prefer dedicated tools bullet)'
description: 'Bulleted warning to prefer dedicated tools over Bash for find, grep, cat, etc.'
ccVersion: 2.1.133
variables:
  - READ_ONLY_SEARCHING_BASH_COMMANDS
-->
- Avoid using this tool to run ${READ_ONLY_SEARCHING_BASH_COMMANDS} commands unless explicitly instructed, or after you've verified a dedicated tool can't accomplish the task. Prefer the dedicated tool — it gives the user a better review experience.
