<!--
name: 'Tool Description: Snooze (delay and reason guidance)'
description: >-
  Extends the snooze tool description with guidance on choosing delaySeconds
  relative to the 5-minute prompt cache TTL and writing informative reason
  fields
ccVersion: 2.1.140
-->
Schedule when to resume work in /loop dynamic mode — the user invoked /loop without an interval, asking you to self-pace iterations of a specific task.

Don't schedule a short-interval wakeup to poll harness-tracked background work you started — you're re-invoked automatically when it finishes, so polling is wasted. Schedule a long fallback (1200s+) so the loop survives if the work hangs or never notifies. The exception is external work the harness can't track (CI, deploy, remote queue) — there, match the delay to how fast that state changes.

Pass the same /loop prompt back via \`prompt\` each turn to repeat the task. For an autonomous /loop (no user prompt), pass the literal sentinel \`${"<<autonomous-loop-dynamic>>"}\` instead — the runtime resolves it at fire time. (There's a similar \`${"<<autonomous-loop>>"}\` sentinel for CronCreate-based autonomous loops; ${"ScheduleWakeup"} always uses the \`-dynamic\` variant.) Omit the call to end the loop.

## Picking delaySeconds

The prompt cache has a 5-minute TTL. Sleeping past 300s reads your context uncached on wake — slower and costlier. Pick by cache window, not round minutes:

- **60s–270s**: cache stays warm. For actively polling external state the harness can't notify you about (CI, deploy, remote queue).
- **300s–3600s**: pay one cache miss. For waiting on something that takes minutes to change, idle, or a long fallback heartbeat behind another primary wake signal.
- **Avoid 300s** — it pays the cache miss without amortizing it. Drop to 270s or commit to 1200s+.

For idle ticks with no specific signal, default to 1200s–1800s. Match the delay to what you're waiting for: a CI run that takes ~8 min wants two ~270s sleeps, not eight 60s ones. The runtime clamps to [60, 3600].

## The reason field

One short, specific sentence on what you chose and why — goes to telemetry and is shown to the user. "watching CI run" beats "waiting."
