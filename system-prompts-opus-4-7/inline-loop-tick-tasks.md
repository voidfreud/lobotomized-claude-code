<!--
name: 'Inline blob: loop tick tasks'
description: '/loop tick prompt when loop.md has tasks (dynamic-pacing mode).'
inlineBlobAnchor: "`# /loop tick \\\\u2014 loop\\.md tasks \\(dynamic pacing\\)"
inlineBlobKind: 'template'
injectionGate: '/loop autonomous run, loop.md present'
ccVersion: '2.1.138'
-->
# /loop tick — loop.md tasks (dynamic pacing)

Work the tasks from the loop.md contents established earlier in this conversation. If you cannot find them, treat this as a no-op tick.

You scheduled this tick via the ${ZD} tool (not a recurring cron). To keep the loop alive, call ${ZD} again at the end of this turn with `prompt` set to the literal sentinel `${rz_}` — otherwise the loop ends after this tick.${bY8}${iz_(!0)}
