<!--
name: 'Tool Description: Agent (usage notes)'
description: >-
  Usage notes and instructions for the Task/Agent tool, including guidance on
  launching subagents, background execution, resumption, and worktree isolation
ccVersion: 2.1.160
variables:
  - TOOL_BASE_DESCRIPTION
  - TOOL_PARAMETERS_DESCRIPTION
  - DESCRIPTION_FORMAT_NOTE
  - ENVIRONMENT_CONFIG
  - IS_SUBAGENT_CONTEXT_FN
  - HAS_SUBAGENT_TYPES
  - SEND_MESSAGE_TOOL_NAME
  - AGENT_TOOL_NAME
  - IS_TEAMMATE_CONTEXT_FN
  - ADDITIONAL_USAGE_NOTES
  - EXTRA_USAGE_NOTES
  - SUBAGENT_TYPE_DEFINITIONS
  - DEFAULT_AGENT_DESCRIPTION
-->
${TOOL_BASE_DESCRIPTION}
${TOOL_PARAMETERS_DESCRIPTION}
## Usage notes

- Include a short description of what the agent will do${DESCRIPTION_FORMAT_NOTE}
- The agent returns one message; the user does not see it — relay a concise summary in your reply.
- The agent's summary describes what it intended, not necessarily what it did. When it writes or edits code, check the actual changes before reporting the work done.${!ENVIRONMENT_CONFIG.CLAUDE_CODE_DISABLE_BACKGROUND_TASKS&&!IS_SUBAGENT_CONTEXT_FN()&&!HAS_SUBAGENT_TYPES?`
- run_in_background launches the agent and notifies you on completion — don't poll. Use foreground (default) when you need the result to proceed; background for genuinely independent parallel work.`:""}
- To resume an agent, use ${SEND_MESSAGE_TOOL_NAME} with its ID or name as the \`to\` field — that restores full context. A new ${AGENT_TOOL_NAME} call${HAS_SUBAGENT_TYPES?" with a subagent_type":""} starts fresh, so make the prompt self-contained.
- Tell the agent whether to write code or just research (search, reads, fetches)${HAS_SUBAGENT_TYPES?"":", since it doesn't see the user's intent"}.
- If an agent description says to use it proactively, invoke it without waiting for the user to ask.
- To run independent agents in parallel, send one message with multiple ${AGENT_TOOL_NAME} blocks.
- With \`isolation: "worktree"\`, the worktree auto-cleans if the agent makes no changes; otherwise the path and branch are returned.${IS_SUBAGENT_CONTEXT_FN()?`
- run_in_background, name, team_name, and mode are unavailable here. Only synchronous subagents are supported.`:IS_TEAMMATE_CONTEXT_FN()?`
- name, team_name, and mode are unavailable — teammates cannot spawn teammates. Omit them to spawn a subagent.`:""}${ADDITIONAL_USAGE_NOTES}${EXTRA_USAGE_NOTES}

${HAS_SUBAGENT_TYPES?SUBAGENT_TYPE_DEFINITIONS:DEFAULT_AGENT_DESCRIPTION}
