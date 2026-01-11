---
# beans-dz8k
title: 'Design: Unified preview panel with tmux mode'
status: completed
type: task
priority: normal
tags:
    - artifact:design
    - needs-review
created_at: 2026-01-05T21:41:17Z
updated_at: 2026-01-05T21:45:59Z
parent: beans-rmtd
---

## Unified Preview Panel with Multiple Modes

Extend the existing preview panel (toggled with `v`) to cycle through three modes:

```
v → v → v
off → body preview → tmux preview → off
```

## State Changes

Replace `showPreview bool` in `App` with a `PreviewMode` enum:

```go
type PreviewMode int
const (
    PreviewOff PreviewMode = iota
    PreviewBody
    PreviewTmux
)
```

The `v` key cycles through the modes. State is global (persists across view changes).

## New Tmux Capture Function

Add to `session.go`:

```go
// CapturePane captures the last N lines from a tmux session's pane
// Uses tmux capture-pane with 1-based line numbering
func CapturePane(sessionName string, lines int) (string, error)
```

Implementation: `tmux capture-pane -t <session>:1 -p -S -<lines>`

## Preview Panel Rendering

Extend the existing preview panel logic:

- **PreviewOff**: No panel, full width for list/detail
- **PreviewBody**: Current behavior (render markdown body)  
- **PreviewTmux**: Display captured pane content (raw text, ~25 lines)
  - If no session exists: show "No session — press a to attach"

## Live Updates

Piggyback on the existing `refreshSessionsMsg` handler (1-second tick):

- When `PreviewTmux` is active, capture pane content for the currently selected bean
- Store/cache captured content to avoid redundant captures
- Only capture when the selected bean has an active session
