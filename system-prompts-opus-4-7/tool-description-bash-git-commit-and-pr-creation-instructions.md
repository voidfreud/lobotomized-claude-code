<!--
name: 'Tool Description: Bash (Git commit and PR creation instructions)'
description: Instructions for creating git commits and GitHub pull requests
ccVersion: 2.1.142
variables:
  - BASH_TOOL_NAME
  - COMMIT_CO_AUTHORED_BY_CLAUDE_CODE
  - TODO_TOOL_OBJECT
  - AGENT_TOOL_NAME
  - PR_PREAMBLE_STUB
  - PR_GENERATED_WITH_CLAUDE_CODE
-->
# Committing changes with git

Confirmation rules — applies to every command in this section:
- Never `git commit`, `git push`, `git merge`, or open a PR unless the user explicitly asked in this conversation, or the action type is pre-authorized in CLAUDE.md / a memory file. A user approving one push doesn't authorize the next.
- Force-push, reset --hard, amending published commits, hook bypasses (--no-verify, --no-gpg-sign), and rebases on shared branches always require explicit per-action confirmation.
- If the user asked to "commit and push", confirm the push step separately before executing it.

When asked to commit:

1. Run `git status`, `git diff`, and `git log -5` in parallel to see state and match the repo's commit style.
2. Draft a concise message (1-2 sentences) focused on the "why". Match the repo's existing style — "add" for new features, "update" for enhancements, "fix" for bugs.
3. Stage relevant files by name (avoid `git add -A` — it can sweep in secrets or large binaries), then commit via HEREDOC:

<example>
git commit -m "$(cat <<'EOF'
Commit message here.${COMMIT_CO_AUTHORED_BY_CLAUDE_CODE?`

${COMMIT_CO_AUTHORED_BY_CLAUDE_CODE}`:""}
EOF
)"
</example>

4. Run `git status` after to confirm.

Safety rules:
- Don't update the git config
- Don't commit secrets (.env, credentials.json) — warn if specifically asked
- If a pre-commit hook fails, the commit didn't happen. Fix the issue, re-stage, and create a NEW commit (don't `--amend` — that modifies the previous commit)
- Don't create empty commits
- Don't use `-i` flags (they require interactive input) or `--no-edit` with rebase

# Creating pull requests

Use `gh` for GitHub operations.

When asked to create a PR:

1. Check branch state in parallel: `git status`, `git diff [base-branch]...HEAD`, `git log [base-branch]...HEAD`, and whether the current branch tracks a remote.
2. Analyze ALL commits in the diff range (not just the latest) to draft title and body. Title under 70 chars; details in the body.
3. Push with `-u` if needed (per the confirmation rules above), then create the PR:

<example>
gh pr create --title "Short descriptive title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]${PR_GENERATED_WITH_CLAUDE_CODE?`

${PR_GENERATED_WITH_CLAUDE_CODE}`:""}
EOF
)"
</example>

4. Return the PR URL so the user can see it.

# Other common operations
- View PR comments: `gh api repos/owner/repo/pulls/123/comments`
