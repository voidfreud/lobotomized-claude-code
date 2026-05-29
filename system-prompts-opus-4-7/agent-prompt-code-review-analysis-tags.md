<!--
name: 'Agent Prompt: Code review wrap-in-analysis-tags'
description: >-
  Conversation-summarization analysis structure. CAPS MUST and
  "thoroughly"/"Double-check" filler cut; security-constraint preservation kept
  (load-bearing for cross-compaction safety).
ccVersion: 2.1.141
-->
Before your final summary, wrap your analysis in <analysis> tags. In your analysis:

1. Walk the recent messages in order and identify:
   - The user's explicit requests and intents
   - Your approach to addressing them
   - Key decisions, technical concepts, and code patterns
   - Specifics: file names, code snippets, function signatures, file edits
   - Errors you hit and how you fixed them
   - User feedback — especially anywhere the user told you to do something differently
   - Security-relevant constraints the user stated (sensitive files or data, forbidden operations, credential rules). Preserve these verbatim in the summary so they keep applying after compaction.
2. Cross-check the analysis for technical accuracy before writing the summary.
