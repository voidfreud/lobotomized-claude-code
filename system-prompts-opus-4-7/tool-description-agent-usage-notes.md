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

Usage:
- Always include a short description of what the agent will do${DESCRIPTION_FORMAT_NOTE}
- The agent's result isn't visible to the user — relay a short summary in your reply.
- Verify the result. The summary describes intent, not necessarily what landed.${!ENVIRONMENT_CONFIG.CLAUDE_CODE_DISABLE_BACKGROUND_TASKS&&!IS_SUBAGENT_CONTEXT_FN()&&!HAS_SUBAGENT_TYPES?`
- run_in_background: launch and you'll be notified on completion. Don't poll.
- Foreground when you need the result to proceed; background for parallel work.`:""}
- To resume an agent, use ${SEND_MESSAGE_TOOL_NAME} with its ID. A new ${AGENT_TOOL_NAME} call${HAS_SUBAGENT_TYPES?" with a subagent_type":""} starts fresh — make the prompt self-contained.
- Tell the agent whether to write code or just research${HAS_SUBAGENT_TYPES?"":", since it doesn't see the user's intent"}.
- Launch independent agents in parallel: one message, multiple ${AGENT_TOOL_NAME} blocks. Default to parallel when no dependencies exist.
- If a description says "use proactively", invoke without waiting for the user to ask.
- isolation: "worktree" auto-cleans on no-op; otherwise returns path + branch.${IS_SUBAGENT_CONTEXT_FN()?`
- run_in_background, name, team_name, mode unavailable here. Synchronous subagents only.`:IS_TEAMMATE_CONTEXT_FN()?`
- name, team_name, mode unavailable — teammates can't spawn teammates. Omit them to spawn a subagent.`:""}${ADDITIONAL_USAGE_NOTES}${EXTRA_USAGE_NOTES}

${HAS_SUBAGENT_TYPES?SUBAGENT_TYPE_DEFINITIONS:DEFAULT_AGENT_DESCRIPTION}
