---
# beans-z3a5
title: Refactor View/ViewWithStatus duplication
status: completed
type: task
priority: high
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:25Z
updated_at: 2026-01-01T22:05:27Z
parent: beans-kwuv
---

## Problem

The `View()` and `ViewWithStatus()` methods in both `features.go` and `detail.go` duplicate significant rendering logic (~100 lines each).

## Files
- `superbeans/internal/tui/features.go:211-344`
- `superbeans/internal/tui/detail.go:180-258`

## Why It Matters
Violates DRY principle. Any rendering changes need to be made in two places, increasing maintenance burden and risk of inconsistency.

## Fix
Refactor so `View()` delegates to `ViewWithStatus("")`:

```go
func (m featuresModel) View() string {
    return m.ViewWithStatus("")
}
```

Apply the same pattern to `detailModel`.