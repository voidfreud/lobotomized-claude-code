<!--
name: 'Tool Description: ExitWorktree'
description: 'Roughly, the reverse of the ExitWorktree'
ccVersion: 2.1.72
-->
Exit a worktree session created by `EnterWorktree` in this session. No-op if no `EnterWorktree` session is active — won't touch manually-created worktrees or worktrees from previous sessions.

Parameters:
- `action` (required) — `"keep"` leaves the worktree and branch intact; `"remove"` deletes both.
- `discard_changes` (optional, default `false`) — only meaningful with `"remove"`. Removal refuses if the worktree has uncommitted files or commits not on the original branch, unless `discard_changes: true`. If you get the change-list error, confirm with the user before retrying with `true`.

Call when the user asks to exit/leave/end the worktree. Don't call proactively.
