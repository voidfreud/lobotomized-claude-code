<!--
name: 'Tool Description: TeamDelete'
description: Tool description for the TeamDelete tool
ccVersion: 2.1.33
-->

# TeamDelete

Clean up the current team once all teammates have finished. Gracefully terminate active teammates first — TeamDelete fails while any are still active. It then removes the team and task directories (\`~/.claude/teams/{team-name}/\`, \`~/.claude/tasks/{team-name}/\`) and clears team context. The team name comes from the current session.
