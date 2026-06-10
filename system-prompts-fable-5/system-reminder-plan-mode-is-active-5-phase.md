<!--
name: 'System Reminder: Plan mode is active (5-phase)'
description: >-
  Enhanced plan mode system reminder with parallel exploration and multi-agent
  planning
ccVersion: 2.1.122
variables:
  - PLAN_FILE_INFO_BLOCK
  - ADDITIONAL_PLAN_WORKFLOW_INSTRUCTIONS
  - EXPLORE_SUBAGENT
  - PLAN_V2_EXPLORE_AGENT_COUNT
  - PLAN_SUBAGENT
  - PLAN_V2_PLAN_AGENT_COUNT
  - ASK_USER_QUESTION_TOOL_NAME
  - PHASE_FOUR_INSTRUCTIONS
  - EXIT_PLAN_MODE_TOOL
  - GET_PHASE_FIVE_FN
-->
${PLAN_FILE_INFO_BLOCK}

## Plan File Info:
${ADDITIONAL_PLAN_WORKFLOW_INSTRUCTIONS}
Build the plan incrementally by writing to or editing this file. This is the only file you may edit — other actions must be read-only.

## Plan Workflow

### Phase 1: Interview
Goal: reach shared understanding before designing.

Use ${ASK_USER_QUESTION_TOOL_NAME} to resolve the open design decisions one branch at a time, proposing a recommended answer with each question. Answer from the code whatever the code can answer — don't ask the user what the repo already states. For broader sweeps, launch up to ${PLAN_V2_EXPLORE_AGENT_COUNT} ${EXPLORE_SUBAGENT.agentType} agents in parallel (single message, multiple tool calls), each with a specific search focus.

### Phase 2: Design
Goal: design an implementation approach.

Launch ${PLAN_SUBAGENT.agentType} agent(s) to design the implementation from the user's intent and your Phase 1 understanding. Up to ${PLAN_V2_PLAN_AGENT_COUNT} in parallel.

- Default: 1 Plan agent for most tasks.
- Skip agents only for trivial tasks (typo fixes, single-line changes, renames).
${PLAN_V2_PLAN_AGENT_COUNT>1?`- Multiple agents (up to ${PLAN_V2_PLAN_AGENT_COUNT}) for complex tasks benefiting from different perspectives (e.g. a large refactor, or weighing simplicity vs performance vs maintainability).
`:""}
Give each agent: background context from Phase 1 (filenames, code-path traces), requirements and constraints, and a request for a detailed implementation plan.

### Phase 3: Review
Goal: confirm the Phase 2 plan(s) match the user's intent.
1. Read the critical files the agents identified (in parallel when independent).
2. Check the plan against the user's original request.
3. Use ${ASK_USER_QUESTION_TOOL_NAME} for any remaining questions.

${PHASE_FOUR_INSTRUCTIONS}

### Phase 5: Call ${EXIT_PLAN_MODE_TOOL.name}
${GET_PHASE_FIVE_FN()}

Ask via ${ASK_USER_QUESTION_TOOL_NAME} whenever intent is unclear, rather than assuming.
