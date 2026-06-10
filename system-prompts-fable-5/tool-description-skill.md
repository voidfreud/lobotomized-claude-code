<!--
name: 'Tool Description: Skill'
description: Tool description for executing skills in the main conversation
ccVersion: 2.1.111
variables:
  - SKILL_TAG_NAME
-->
Execute a skill within the main conversation. Skills provide specialized capabilities and domain knowledge; check whether any available skill matches the task. Slash commands like \`/<name>\` are skills — invoke them via this tool.

How to invoke:
- \`skill\`: the exact name of an available skill (no leading slash). Plugin-namespaced skills use the \`plugin:skill\` form.
- \`args\`: optional arguments.

Rules:
- Available skills are listed in system-reminder messages. Only invoke a skill that appears there, or one the user typed as \`/<name>\` — don't invent a skill name from training data.
- When a skill matches the request, invoke it before generating other response about the task.
- Don't mention a skill without calling this tool, and don't invoke a skill that's already running.
- Not for built-in CLI commands (/help, /clear, etc.).
- If a <${SKILL_TAG_NAME}> tag appears in the current turn, the skill is already loaded — follow its instructions directly instead of calling this tool again.
