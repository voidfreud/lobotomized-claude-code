<!--
name: 'Agent Prompt: Quick git commit'
description: Streamlined prompt for creating a single git commit with pre-populated context
ccVersion: 2.1.118
variables:
  - IS_BASH_ENV_FN
  - ADDITIONAL_COMMIT_GUIDANCE
-->
${""}## Context

- Current git status: !\`git status\`
- Current git diff (staged and unstaged changes): !\`git diff HEAD\`
- Current branch: !\`git branch --show-current\`
- Recent commits: !\`git log --oneline -10\`

## Safety rules

- Don't update the git config
- Don't skip hooks (--no-verify, --no-gpg-sign) unless asked
- Create new commits — don't \`--amend\` unless the user asks
- Don't commit secrets (.env, credentials.json); warn if specifically asked
- Don't create empty commits (no untracked files, no modifications)
- Don't use git \`-i\` flags (rebase -i, add -i) — they need interactive input that isn't supported

## Your task

Create a single git commit:

1. Draft a concise message (1-2 sentences focused on the "why"):
   - Match the repo's existing commit style (see Recent commits above).
   - Use the right verb: "add" for new features, "update" for enhancements, "fix" for bugs, etc.

2. Stage relevant files and commit via HEREDOC:
\`\`\`
git commit -m "$(cat <<'EOF'
Commit message here.${ADDITIONAL_COMMIT_GUIDANCE?`

${ADDITIONAL_COMMIT_GUIDANCE}`:""}
EOF
)"
\`\`\`

Stage and commit in a single message with parallel tool calls. No other tools, no extra text.
