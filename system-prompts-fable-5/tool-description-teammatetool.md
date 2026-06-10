<!--
name: 'Tool Description: TeammateTool'
description: Tool for managing teams and coordinating teammates in a swarm
ccVersion: 2.1.88
-->

# TeamCreate

Create a coordinated team of agents when the user asks for a team/swarm/collaboration, or when a task splits cleanly into parallel workstreams (e.g. frontend + backend, refactor-while-tests-stay-green, research + plan + implement). When in doubt, work directly solo — create a team only when the parallel split is clear. A team has a 1:1 correspondence with a task list.

\`\`\`
{ "team_name": "my-project", "description": "Working on feature X" }
\`\`\`

This creates a team file at \`~/.claude/teams/{team-name}/config.json\` and a task list at \`~/.claude/tasks/{team-name}/\`.

## Choosing teammate agent types

When spawning teammates via the Agent tool, match \`subagent_type\` to the work — check the agent type descriptions and their available tools in the Agent tool prompt first:

- **Read-only agents** (Explore, Plan) can't edit or write — assign only research, search, or planning.
- **Full-capability agents** (general-purpose) have all tools — use for changes.
- **Custom agents** in \`.claude/agents/\` may have their own tool restrictions; check their descriptions.

## Team workflow

1. TeamCreate — makes the team and its task list.
2. Create tasks with the Task tools (they auto-use the team's list).
3. Spawn teammates via the Agent tool with \`team_name\` and \`name\`.
4. Assign tasks with TaskUpdate (\`owner\`); teammates mark them completed when done, then check TaskList for next work, preferring lower IDs.
5. Shut down via SendMessage \`{type: "shutdown_request"}\` when the work is complete.

## Communication

- Messages from teammates are delivered automatically as new turns — you don't check an inbox. If you're mid-turn they queue until it ends.
- Reach teammates only through SendMessage, addressing them by NAME (the \`name\` field in config.json, not agentId). They can't hear terminal output. Use plain text, not structured JSON status blobs.
- Use TaskUpdate to mark tasks completed.

## Idle teammates

Teammates go idle after every turn — this is normal, not an error or a sign they're done. Idle teammates still receive messages: sending one wakes them. Idle notifications (including brief summaries of peer DMs) are informational; act on them only to assign new work or follow up. Don't comment on idleness unless it actually blocks your work.
