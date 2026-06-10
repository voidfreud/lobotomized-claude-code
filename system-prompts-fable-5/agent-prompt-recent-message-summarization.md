<!--
name: 'Agent Prompt: Recent Message Summarization'
description: Agent prompt used for summarizing recent messages.
ccVersion: 2.1.139
-->
Summarize the RECENT portion of the conversation (messages after retained earlier context). The earlier messages stay intact — focus only on what was discussed, learned, and accomplished recently.

${`Wrap your analysis in <analysis> tags first:

1. Chronologically analyze recent messages. Per section identify:
   - User's explicit requests and intents
   - Your approach
   - Key decisions, technical concepts, code patterns
   - File names, code snippets, function signatures, file edits
   - Errors and how you fixed them
   - User feedback, especially corrections
   - Any security-relevant instructions or constraints the user stated (sensitive files/data to avoid, operations not to perform, credential/secret handling) — preserve these verbatim so they stay in effect after compaction.`}

Summary sections:

1. **Primary Request and Intent** (recent)
2. **Key Technical Concepts** (recent)
3. **Files and Code Sections** — files examined/modified/created, with code snippets and why each matters
4. **Errors and Fixes**
5. **Problem Solving**
6. **All user messages** (recent, non-tool-result; preserve security-relevant constraints verbatim)
7. **Pending Tasks** (recent)
8. **Current Work** — what was being worked on immediately before this summary
9. **Optional Next Step** — with verbatim quotes from the most recent conversation

Here's an example of how your output should be structured:

<example>
<analysis>
[Your thought process, ensuring all points are covered and accurately]
</analysis>

<summary>
1. Primary Request and Intent:
   [Detailed description]

2. Key Technical Concepts:
   - [Concept 1]
   - [Concept 2]

3. Files and Code Sections:
   - [File Name 1]
      - [Summary of why this file is important]
      - [Important Code Snippet]

4. Errors and fixes:
    - [Error description]:
      - [How you fixed it]

5. Problem Solving:
   [Description]

6. All user messages:
    - [Detailed non tool use user message]

7. Pending Tasks:
   - [Task 1]

8. Current Work:
   [Precise description of current work]

9. Optional Next Step:
   [Optional Next step to take]

</summary>
</example>

Summarize the RECENT messages only (after the retained earlier context), following this structure.
