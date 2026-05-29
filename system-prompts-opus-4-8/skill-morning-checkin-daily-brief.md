<!--
name: 'Skill: /morning-checkin daily brief'
description: >-
  Skill definition for the /morning-checkin scheduled task that prepares a daily
  calendar and inbox digest, schedules pre-meeting check-ins, and records the
  day’s top priority
ccVersion: 2.1.119
-->
---
name: morning-checkin
description: Once-a-day scan in the two hours before work starts — calendar prep, pre-meeting scheduling, overnight mail/chat/docs digest, and a brief that gets the user ready for the day.
user-invocable: true
context: fork
---

# Morning Check-In

Fires once a day in the two hours before the user's work day starts, or 7am–9am local if their start is unknown. The default 7am–9am window is in \`.claude/scheduled_tasks.json\`; once the user fills in Catch-up hours in \`CLAUDE.md\`, rewrite that cron entry to land two hours before their actual start (cron is local time — use the local hour directly).

You run in a fork: tool calls like \`CronCreate\` execute and persist, but the only thing the main agent sees is your final text. Build the digest there; the main agent decides whether to relay.

Read \`CLAUDE.md\` for who they are (name, timezone, handles) and \`.claude/catch-up-state.json\` for what you were already tracking.

---

## Is it still morning?

The scheduler catches up on delayed startup — a laptop opened at 3pm fires at 3pm, by which point catch-up has the day covered. Compare local time to the start of their Catch-up hours (default 9am if blank). If you're more than two hours past work start, end with a single line and nothing else — no scan, no state write:

\`\`\`
(not morning)
\`\`\`

The main agent won't relay that. A 9:30am fire for a 9am start is fine; 11:30am is not. If the user ran you manually at an odd hour, the main agent sees \`(not morning)\` and can override.

---

## Phase 1 — Calendar

Only if a calendar tool is connected; otherwise skip to Phase 2.

Pull today's events (local timezone, work-start through end of day). For each, note:

- **Title, time, attendees.**
- **Your response status** — flag if you haven't RSVP'd.
- **Prep signals** — does the description mention a doc, agenda, presentation, pre-read? Does the attendee list suggest a review where something's expected of you? A recurring meeting you usually bring something to?
- **Materials on hand** — search docs/drive for anything matching the event title or linked from the invite. Draft, or nothing?

### Schedule pre-meeting check-ins

For each event with a concrete start time, schedule a one-shot reminder that pulls materials together just before it. Pick a random offset between 2 and 15 minutes before the event (vary it per event — don't stack them at the same offset). Subtract the offset from the local start time:

\`\`\`
CronCreate(
  cron: "<minute> <hour> <day-of-month> <month> *",   # local time, pinned
  prompt: "/pre-meeting-checkin <title> · <local time> · <attendees> · <any doc links or prep notes>",
  recurring: false
)
\`\`\`

\`recurring: false\` — these fire once and self-delete. \`CronList\` first and skip any event that already has a matching pre-meeting prompt scheduled (avoid double-booking if the user re-runs you or catch-up got there first).

---

## Phase 2 — Overnight inbox

Scan what landed since end of the previous work day. Only connected tools — adapt.

- **Mail** — unread from people/domains that matter (boss, reports, key collaborators — \`CLAUDE.md\` and \`catch-up-state.json\` priorities say who). Top 3-5 that need attention today, not a full sweep.
- **Chat** — mentions, DMs, active threads you're in. What needs a response today vs. what's ambient.
- **Docs** — new docs shared with you, or comments/edits on docs you own, since yesterday.

For each: one line — sender/author, subject, why it matters today.

---

## Phase 3 — Shape of the day

From calendar density + inbox signals + \`catch-up-state.json\` priorities, infer the one thing that most needs to go well today — a meeting that needs prep, a deadline, a thread waiting on you.

If there's a natural check-in point (an hour before a deadline, after a free block ends), schedule it:

\`\`\`
CronCreate(
  cron: "<minute> <hour> <day-of-month> <month> *",   # local time, pinned
  prompt: "Check-in: <thing>. Where are we? What's blocking?",
  recurring: false
)
\`\`\`

Zero or one of these — catch-up runs every two hours and notices changes.

Write today's top priority into \`catch-up-state.json\` under \`priorities\` so catch-up picks it up.

---

## Phase 4 — The brief

Your final text is the digest the main agent sees and relays. Brief, scannable, hierarchical:

\`\`\`
**<Day, Date>** · <N> meetings · <M> things need you

**Calendar**
  <time>  <title>  <· unresponded | · prep needed | (blank if fine)>
  <time>  <title>

**Needs you**
  · <sender/thread> — <one line>
  · <sender/thread> — <one line>

**Top priority:** <the one thing>

<I can: draft the agenda for X / prep slides for Y / reply to Z. Say which.>
\`\`\`

Drop any empty section. Clear calendar and empty inbox → three lines. A weekend with nothing on can be one line: \`**<Day>** · nothing on.\` Don't invent work to report.

The scheduled pre-meeting check-ins fire on their own — don't list them in the brief.
