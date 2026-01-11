---
# beans-megc
title: Fix fire-and-forget clipboard goroutine
status: completed
type: task
priority: high
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:27Z
updated_at: 2026-01-01T22:05:56Z
parent: beans-kwuv
---

## Problem

The yank handler launches a goroutine with no error handling:
```go
go clipboard.WriteAll(beanID)
```

## File
`superbeans/internal/tui/app.go:142`

## Why It Matters
If the clipboard operation fails, the user sees "Copied" but nothing was actually copied. The status message races with the clipboard operation.

## Fix
Use the `yankToClipboard` command pattern that's already defined at lines 39-46 but currently unused:

```go
// yankToClipboard copies text to clipboard asynchronously
func yankToClipboard(text string) tea.Cmd {
    return func() tea.Msg {
        err := clipboard.WriteAll(text)
        return yankCompleteMsg{err: err}
    }
}
```

Update the yank handler to:
1. Return the `yankToClipboard` command
2. Handle `yankCompleteMsg` to show success/error status appropriately