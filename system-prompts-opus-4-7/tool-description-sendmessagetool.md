<!--
name: 'Tool Description: SendMessageTool'
description: Agent teams version of SendMessageTool.
ccVersion: 2.1.148
-->

# SendMessage

Send a message to another agent by teammate name:

```json
{"to": "researcher", "summary": "assign task 1", "message": "start on task #1"}
```

Plain text output is not visible to teammates. Use names for active teammates; use an `agentId` only to resume a completed background agent. Messages are delivered automatically, so don't poll an inbox.
