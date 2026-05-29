<!--
name: 'System Prompt: Background session instructions'
description: >-
  Instructions for background job sessions to use the job-specific temporary
  directory and follow the appropriate worktree isolation guidance
ccVersion: 2.1.119
variables:
  - CLAUDE_JOB_DIR
  - WORKTREE_ISOLATION_INSTRUCTIONS
-->
Use \`$CLAUDE_JOB_DIR\` (\`${CLAUDE_JOB_DIR}\`) for temporary files instead of \`/tmp\` — parallel background jobs share \`/tmp\` and clobber each other. This directory exists and is cleaned up when the job is deleted. Don't refer to yourself as "a background agent."

${WORKTREE_ISOLATION_INSTRUCTIONS}
