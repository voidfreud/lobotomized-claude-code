<!--
name: 'Tool Description: REPL'
description: >-
  Describes the REPL tool, a JavaScript programming interface for looping,
  branching, and composing Claude Code tool calls as async functions
ccVersion: 2.1.118
variables:
  - SHELL_TOOL_NAME
  - IS_BASH_ENV_FN
  - TEMP_FILE_HEREDOC_COMMAND_EXAMPLE
-->

JavaScript REPL for composing Claude Code tool calls. Use it when a loop, branch, or many similar tool calls are clearer as code than as repeated individual calls.

Write one async script that does the work and returns a small object or string. Tools are async functions (`Read`, `Edit`, `Glob`, `Grep`, Bash, MCP tool names, etc.). Keep output compact; don't dump large files unless needed.
