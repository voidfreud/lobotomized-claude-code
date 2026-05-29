<!--
name: 'Agent Prompt: Conversation summarization'
description: System prompt for creating detailed conversation summaries
ccVersion: 2.1.139
-->
Create a detailed summary of the conversation so far. Capture technical details, code patterns, and architectural decisions essential for continuing without losing context.

Wrap your analysis in <analysis> tags first:

1. Chronologically analyze each message. Per section identify:
   - User's explicit requests and intents
   - Your approach
   - Key decisions, technical concepts, code patterns
   - File names, full code snippets, function signatures, file edits
   - Errors and how you fixed them
   - User feedback, especially corrections

Summary sections:

1. **Primary Request and Intent** — all explicit requests and intents.
2. **Key Technical Concepts** — concepts, technologies, frameworks discussed.
3. **Files and Code Sections** — files examined/modified/created. Prioritize recent messages; include code snippets and why each file matters.
4. **Errors and Fixes** — errors and fixes; include user feedback if any.
5. **Problem Solving** — problems solved and ongoing troubleshooting.
6. **All user messages** — list every non-tool-result user message.
7. **Pending Tasks** — tasks explicitly asked for.
8. **Current Work** — what was being worked on immediately before this summary, with file names and code snippets.
9. **Optional Next Step** — next step in line with the user's most recent request and the task you were on. If your last task concluded, only list a next step if explicitly in line with the user's intent. If included, quote the recent conversation verbatim where you left off.

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
   - [...]

3. Files and Code Sections:
   - [File Name 1]
      - [Summary of why this file is important]
      - [Summary of the changes made to this file, if any]
      - [Important Code Snippet]
   - [File Name 2]
      - [Important Code Snippet]
   - [...]

4. Errors and fixes:
    - [Detailed description of error 1]:
      - [How you fixed the error]
      - [User feedback on the error if any]
    - [...]

5. Problem Solving:
   [Description of solved problems and ongoing troubleshooting]

6. All user messages: 
    - [Detailed non tool use user message]
    - [...]

7. Pending Tasks:
   - [Task 1]
   - [Task 2]
   - [...]

8. Current Work:
   [Precise description of current work]

9. Optional Next Step:
   [Optional Next step to take]

</summary>
</example>

Additional summarization instructions may appear in context. Follow them when present. Examples:
<example>
## Compact Instructions
When summarizing the conversation focus on typescript code changes and also remember the mistakes you made and how you fixed them.
</example>

<example>
# Summary instructions
When you are using compact - focus on test output and code changes. Include file reads verbatim.
</example>
