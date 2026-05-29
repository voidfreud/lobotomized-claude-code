<!--
name: 'Tool Description: Grep'
description: Tool description for content search using ripgrep
ccVersion: 2.0.14
variables:
  - GREP_TOOL_NAME
  - BASH_TOOL_NAME
  - TASK_TOOL_NAME
-->
Search file contents with ripgrep.

- Use ${GREP_TOOL_NAME} instead of bash rg/grep — correct permissions.
- Full regex syntax. Filter with glob (e.g. "*.tsx") or type (e.g. "py").
- Output modes: "content" (matching lines), "files_with_matches" (default), "count".
- Literal braces need escaping in ripgrep: `interface\{\}`.
- For multi-line patterns (e.g. `struct \{[\s\S]*?field`), set multiline: true.
- Run independent greps for different patterns in parallel — one message, multiple calls.
- Use ${TASK_TOOL_NAME} for open-ended, multi-round searches.
