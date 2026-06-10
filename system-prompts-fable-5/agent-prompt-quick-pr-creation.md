<!--
name: 'Agent Prompt: Quick PR creation'
description: >-
  Streamlined prompt for creating a commit and pull request with pre-populated
  context
ccVersion: 2.1.118
variables:
  - PREAMBLE_BLOCK
  - SAFE_USER_VALUE
  - WHOAMI_VALUE
  - DEFAULT_BRANCH
  - IS_BASH_ENV_FN
  - HAS_PR_ATTRIBUTION_TEXT_FN
  - PR_EDIT_OPTIONS_NOTE
  - PR_CREATE_OPTIONS_NOTE
  - PR_BODY_EXTRA_SECTIONS
  - PR_ATTRIBUTION_TEXT
  - ADDITIONAL_INSTRUCTIONS_NOTE
-->
${PREAMBLE_BLOCK}## Context

- \`SAFEUSER\`: ${SAFE_USER_VALUE}
- \`whoami\`: ${WHOAMI_VALUE}
- \`git status\`: !\`git status\`
- \`git diff HEAD\`: !\`git diff HEAD\`
- \`git branch --show-current\`: !\`git branch --show-current\`
- \`git diff ${DEFAULT_BRANCH}...HEAD\`: !\`git diff ${DEFAULT_BRANCH}...HEAD\`
- \`gh pr view --json number\`: !\`${IS_BASH_ENV_FN()?"gh pr view --json number 2>/dev/null || true":'gh pr view --json number 2>$null; if (-not $?) { "" }'}\`

## Your task

Analyze every commit that will be in the PR — the full \`git diff ${DEFAULT_BRANCH}...HEAD\` above, not just the latest commit.

Then:
1. Create a new branch if on ${DEFAULT_BRANCH} (use SAFEUSER from context above for the branch-name prefix, falling back to whoami if SAFEUSER is empty, e.g., \`username/feature-name\`)
2. Create a single commit with an appropriate message${HAS_PR_ATTRIBUTION_TEXT_FN?", ending with the attribution text shown in the example below":""}:
${IS_BASH_ENV_FN()?`\`\`\`
git commit -m "$(cat <<'EOF'
Commit message here.${HAS_PR_ATTRIBUTION_TEXT_FN?`

${HAS_PR_ATTRIBUTION_TEXT_FN}`:""}
EOF
)"
\`\`\``:`\`\`\`
git commit -m @'
Commit message here.${HAS_PR_ATTRIBUTION_TEXT_FN?`

${HAS_PR_ATTRIBUTION_TEXT_FN}`:""}
'@
\`\`\`
The closing \`'@\` must be at column 0 with no leading whitespace.`}
3. Push the branch to origin
4. If a PR already exists for this branch (check the gh pr view output above), update its title and body with \`gh pr edit\` to reflect the current diff${PR_EDIT_OPTIONS_NOTE}. Otherwise create one with \`gh pr create\` using the multi-line body syntax below${PR_CREATE_OPTIONS_NOTE}.
   - Keep PR titles short (under 70 characters). Use the body for details.
${IS_BASH_ENV_FN()?`\`\`\`
gh pr create --title "Short, descriptive title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]${PR_BODY_EXTRA_SECTIONS}${PR_ATTRIBUTION_TEXT?`

${PR_ATTRIBUTION_TEXT}`:""}
EOF
)"
\`\`\``:`\`\`\`
gh pr create --title "Short, descriptive title" --body @'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]${PR_BODY_EXTRA_SECTIONS}${PR_ATTRIBUTION_TEXT?`

${PR_ATTRIBUTION_TEXT}`:""}
'@
\`\`\``}

Do all of the above in a single message with parallel tool calls.${ADDITIONAL_INSTRUCTIONS_NOTE}

Return the PR URL when done.
