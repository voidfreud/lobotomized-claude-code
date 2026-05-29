<!--
name: 'Tool Description: EnterWorktree'
description: Tool description for the EnterWorktree tool.
ccVersion: 2.1.133
-->
Create a git worktree and switch the session into it. Use only when the user explicitly says "worktree" or CLAUDE.md instructs it — for new branches without isolation, use `git checkout -b`.

Behavior in a git repo: new worktree under `.claude/worktrees/` on a new branch, based on the `worktree.baseRef` setting (`fresh` = origin/<default-branch>, `head` = current HEAD). Outside a git repo: delegates to `WorktreeCreate` / `WorktreeRemove` hooks.

Parameters:
- `name` — name for a new worktree (random if omitted).
- `path` — enter an existing worktree of this repo (must appear in `git worktree list`). Mutually exclusive with `name`. `ExitWorktree` won't delete a worktree entered this way; use `action: "keep"` to leave.
