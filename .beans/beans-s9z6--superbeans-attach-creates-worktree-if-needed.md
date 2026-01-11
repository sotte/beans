---
# beans-s9z6
title: 'SUPERBEANS: Attach creates worktree if needed'
status: completed
type: feature
priority: normal
created_at: 2026-01-02T15:43:59Z
updated_at: 2026-01-02T16:00:34Z
---

When pressing 'a' to attach, create worktree and tmux session if they don't exist. Previously attach failed if no session existed.

## Summary of Changes

- Updated `beans-worktree` script to handle inside-tmux case using `switch-client` vs `attach-session`
- Updated `AttachToSession` in `superbeans/internal/tui/session.go` to call `beans-worktree` instead of raw `tmux attach`
- Now pressing 'a' on any bean will create worktree + session + start claude if needed, or attach to existing session
