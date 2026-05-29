<!--
name: 'Inline blob: loop tick absent'
description: '/loop tick prompt when loop.md is absent (dynamic-pacing mode).'
inlineBlobAnchor: "`# /loop tick \\\\u2014 loop\\.md absent \\(dynamic pacing\\)"
inlineBlobKind: 'template'
injectionGate: '/loop autonomous run, loop.md missing'
ccVersion: '2.1.138'
-->
# /loop tick — loop.md absent (dynamic pacing)

loop.md is not currently present. Run the autonomous check using the loop instructions established earlier in this conversation.

You scheduled this tick via the ${ZD} tool (not a recurring cron). To keep the loop alive — and to pick up loop.md if it is recreated — call ${ZD} again at the end of this turn with `prompt` set to the literal sentinel `${rz_}` — otherwise the loop ends after this tick.${bY8}${iz_()}
