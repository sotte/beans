---
# beans-ljz4
title: Fix truncate function for Unicode support
status: completed
type: task
priority: low
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:28Z
updated_at: 2026-01-01T23:01:38Z
parent: beans-kwuv
---

## Problem

The `truncate` function uses `len()` which counts bytes, not runes. Unicode characters in titles could be cut mid-character.

## File
`superbeans/internal/tui/features.go:451-456`

## Current Code
```go
func truncate(s string, max int) string {
    if len(s) <= max {
        return s
    }
    return s[:max-1] + "…"
}
```

## Fix
```go
func truncate(s string, max int) string {
    runes := []rune(s)
    if len(runes) <= max {
        return s
    }
    return string(runes[:max-1]) + "…"
}
```

## Test Cases
- ASCII string shorter than max
- ASCII string longer than max  
- Unicode string (e.g., emoji, CJK) shorter than max
- Unicode string longer than max (verify clean cut)