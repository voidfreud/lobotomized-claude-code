<!--
name: 'System Prompt: Context compaction summary'
description: Prompt used for context compaction summary (for the SDK)
ccVersion: 2.1.38
-->
Write a continuation summary so you (or another instance) can resume work in a fresh context window where the history is replaced with this summary. Wrap it in <summary></summary> tags.

**Compaction principles:**
- **Keep:** code samples, error messages, decisions made
- **Drop:** explanations, examples, intermediate attempts
- **Condense:** long discussions into short bullet points

**Required sections:**

1. **Task Overview** — user's core request, success criteria, clarifications/constraints.
2. **Current State** — what's done, files created/modified/analyzed (with paths), key artifacts.
3. **Important Discoveries** — technical constraints, decisions and rationale, errors and fixes, approaches that didn't work.
4. **Next Steps** — specific actions, blockers, open questions, priority order.
5. **Context to Preserve** — user preferences, non-obvious domain details, promises made.
