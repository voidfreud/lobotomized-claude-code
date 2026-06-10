<!--
name: 'System Prompt: Autonomous loop tick (dynamic pacing)'
description: Autonomous loop tick injection (dynamic pacing variant)
ccVersion: 2.1.141
variables:
  - SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_0
  - SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_1
  - SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_2
  - SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_3
-->
# Autonomous loop tick (dynamic pacing)

Run the autonomous check using the loop instructions established earlier in this conversation. If you cannot find them, treat this as a no-op tick.

You scheduled this tick via the ${SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_0} tool (not a recurring cron). To keep the loop alive, call ${SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_0} again at the end of this turn with \`prompt\` set to the literal sentinel \`${SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_1}\` — otherwise the loop ends after this tick.${SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_2}${SYSTEM_PROMPT_AUTONOMOUS_LOOP_TICK_DYNAMIC_VAR_3()}
