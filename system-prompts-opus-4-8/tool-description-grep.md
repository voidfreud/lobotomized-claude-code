<!--
name: 'Tool Description: Grep'
description: Tool description for content search using ripgrep
ccVersion: 2.0.14
variables:
  - GREP_TOOL_NAME
  - BASH_TOOL_NAME
  - TASK_TOOL_NAME
-->
A powerful search tool built on ripgrep.

Usage:
- Use ${GREP_TOOL_NAME} for search tasks rather than \`grep\` or \`rg\` as a ${BASH_TOOL_NAME} command — it's optimized for correct permissions and access.
- Supports full regex syntax (e.g. "log.*Error", "function\\s+\\w+").
- Filter files with the glob parameter (e.g. "*.js", "**/*.tsx") or type parameter (e.g. "js", "py", "rust").
- Output modes: "content" (matching lines), "files_with_matches" (paths only, default), "count" (match counts).
- Pattern syntax uses ripgrep (not grep): literal braces need escaping (\`interface\\{\\}\` to find \`interface{}\` in Go).
- Patterns match within single lines by default; for cross-line patterns (e.g. \`struct \\{[\\s\\S]*?field\`) set \`multiline: true\`.
- Use ${TASK_TOOL_NAME} for open-ended searches requiring multiple rounds.
