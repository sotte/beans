---
# beans-rmtd
title: Show content of attached tmux session
status: in-progress
type: feature
priority: normal
tags:
    - artifact:design
created_at: 2026-01-05T21:22:30Z
updated_at: 2026-01-05T21:42:05Z
---

Display the content/output of the attached tmux session (i.e. the Claude Code session) within the superbeans TUI.

## Motivation

When monitoring agent work via superbeans, it would be useful to see what the Claude session is actually doing without switching to the tmux pane.

## Architecture Overview

Extend the existing preview panel to cycle through three modes with `v`:
- **off** → **body preview** → **tmux preview** → off

Key changes:
- Replace `showPreview bool` with `PreviewMode` enum in `App`
- Add `CapturePane()` function to `session.go` using `tmux capture-pane`
- Extend preview panel rendering for tmux mode
- Piggyback on existing 1-second session poll for live updates
