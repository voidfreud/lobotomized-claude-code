<!--
name: 'Tool Description: Agent (simple usage notes)'
description: >-
  Simplified usage notes for the Agent tool, including when to delegate, fork
  behavior, resumption, worktree isolation, background execution, parallel
  launches, and context restrictions
ccVersion: 2.1.140
variables:
  - TOOL_BASE_DESCRIPTION
  - HAS_PRO_RESTRICTION_NOTE
  - CAN_FORK_CONTEXT
  - SEND_MESSAGE_TOOL_NAME
  - AGENT_TOOL_NAME
  - RUN_IN_BACKGROUND_NOTE
  - PARALLEL_AGENTS_NOTE
  - CONTEXT_RESTRICTION_NOTE
-->
${TOOL_BASE_DESCRIPTION}${HAS_PRO_RESTRICTION_NOTE?"":`

## When to use

Delegate when the task matches an agent type, when you have independent work to run in parallel, or when answering would mean reading across several files — you keep the conclusion, not the file dumps. For a single-fact lookup where you already know the file, symbol, or value, search directly. Once you've delegated a search, wait for the result rather than running it yourself too.`}${CAN_FORK_CONTEXT?`

A fork runs in the background and keeps its tool output out of your context. If you are the fork, execute directly — don't re-delegate.`:""}

- The agent's final message is the tool result; the user does not see it — relay what matters.
- ${SEND_MESSAGE_TOOL_NAME} with the agent's ID or name continues a spawned agent with its context intact; a new ${AGENT_TOOL_NAME} call${CAN_FORK_CONTEXT?" with a subagent_type":""} starts fresh.
- \`isolation: "worktree"\` gives the agent its own git worktree (auto-cleaned if unchanged).${RUN_IN_BACKGROUND_NOTE}${PARALLEL_AGENTS_NOTE}${CONTEXT_RESTRICTION_NOTE}
