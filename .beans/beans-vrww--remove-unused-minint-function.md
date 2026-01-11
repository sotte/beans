---
# beans-vrww
title: Remove unused minInt function
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:27Z
updated_at: 2026-01-01T23:01:03Z
parent: beans-kwuv
---

## Problem

`minInt` is defined but never used.

## File
`superbeans/internal/tui/detail.go:412-417`

## Why It Matters
Dead code adds confusion and should be removed.

## Fix
Delete the function:
```go
func minInt(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```