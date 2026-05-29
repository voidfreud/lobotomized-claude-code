<!--
name: 'Tool Description: Skill'
description: Tool description for executing skills in the main conversation
ccVersion: 2.1.111
variables:
  - SKILL_TAG_NAME
-->
Run a skill in the main conversation.

Slash commands like `/<name>` are skills — invoke via this tool.

- `skill`: exact name from the available list (no leading slash). Plugin skills use `plugin:skill`.
- `args`: optional arguments.

- Only invoke skills listed in system-reminders or one the user typed (`/<name>`). Don't guess from training data.
- When a skill matches the request, invoke it before describing it.
- Don't reinvoke a running skill.
- Don't use this for built-in CLI (/help, /clear, etc.).
- If a <${SKILL_TAG_NAME}> tag is in the current turn, the skill is already loaded — follow it directly, don't recall.
