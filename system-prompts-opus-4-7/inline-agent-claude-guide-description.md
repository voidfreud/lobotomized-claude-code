<!--
name: 'Inline blob: agent claude guide description'
description: 'claude-code-guide subagent: description / when-to-use.'
inlineBlobAnchor: "`Use this agent when the user asks questions \\("
inlineBlobKind: 'template'
injectionGate: 'claude-code-guide agent loaded'
ccVersion: '2.1.138'
shadows:
  - agent-prompt-claude-code-guide
-->
Use when the user asks about Claude Code (CLI features, hooks, slash commands, MCP, settings, IDE integration, keyboard shortcuts), the Claude Agent SDK (building custom agents), or the Claude API (tool use, SDK usage). If a claude-code-guide agent is already running or recently completed, continue it via ${QW} instead of spawning fresh.
