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

## Git safety rules

- Don't update the git config.
- Don't skip hooks (--no-verify, --no-gpg-sign, etc.) unless the user explicitly asks.
- Always create new commits; don't use git commit --amend unless the user explicitly asks.
- Don't commit files likely to contain secrets (.env, credentials.json, etc.). Warn the user if they specifically ask to commit those.
- Don't create an empty commit when there are no untracked files and no modifications.
- Don't use git -i flags (rebase -i, add -i) — they need interactive input, which isn't supported.

## Your task

Create a single git commit:

1. Draft a concise (1-2 sentence) commit message focused on the "why":
   - Match this repo's commit style (see Recent commits above).
   - Use the right verb: "add" for a new feature, "update" for an enhancement, "fix" for a bug, etc.

2. Stage relevant files and create the commit:
${IS_BASH_ENV_FN()?`\`\`\`
git commit -m "$(cat <<'EOF'
Commit message here.${ADDITIONAL_COMMIT_GUIDANCE?`

${ADDITIONAL_COMMIT_GUIDANCE}`:""}
EOF
)"
\`\`\``:`\`\`\`
git commit -m @'
Commit message here.${ADDITIONAL_COMMIT_GUIDANCE?`

${ADDITIONAL_COMMIT_GUIDANCE}`:""}
'@
\`\`\`
The closing \`'@\` MUST be at column 0 with no leading whitespace.`}

Stage and commit using parallel tool calls in a single message. Don't use other tools or send any other text.
