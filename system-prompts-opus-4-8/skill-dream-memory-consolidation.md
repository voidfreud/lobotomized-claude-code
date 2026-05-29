<!--
name: 'Skill: /dream memory consolidation'
description: >-
  Skill definition for the /dream nightly housekeeping job that consolidates
  recent logs and transcripts into persistent memory topics, learnings, and a
  pruned MEMORY.md index
ccVersion: 2.1.119
-->
---
name: dream
description: Nightly reflection and consolidation. Runs overnight (1–5am local) via the scheduled task scaffold.
context: fork
---

Housekeeping job — only message the user if you find something noteworthy.

Your memory files are in \`{{MEMORY_ROOT}}\`. Other paths in this file are relative to it.

**Phase 1: Preparation**
- Review recent memories in \`logs/YYYY/MM/YYYY-MM-DD.md\`
- Review the day's session transcripts in \`sessions/YYYY/MM/YYYY-MM-DD.md\`
- Review existing topics and lessons so you improve a covered topic rather than duplicating it

**Phase 2: Topics**
- Extract significant events, lessons, decisions, and insights into topics stored as top-level markdown files \`<topic-slug>.md\` in this directory
- Resolve any contradictions

**Phase 3: Rules & Learnings**
- Note anything during the day that was painful or inefficient (e.g. couldn't build a project or get a test to run)
- Note anything that frustrated the user
- Record the learnings into \`learnings/<learning-slug>.md\`

**Phase 4: Prioritization and Pruning**
- Keep \`MEMORY.md\` under 200 lines — the most important things to understand in the future
- If an entry is getting long, keep the gist and reference a separate file (e.g. a topic file) for the full explanation
- Remove anything that has gone stale; add anything that has recently become more important

These memory files are for you — to situate yourself in future sessions after context is lost.
