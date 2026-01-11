---
# beans-6k4c
title: Extract done limit magic number to constant
status: scrapped
type: task
priority: low
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:29Z
updated_at: 2026-01-01T23:00:28Z
parent: beans-kwuv
---

## Problem

The number `5` (limit of done items shown) is repeated in multiple places.

## Files
- `superbeans/internal/tui/features.go:174`
- `superbeans/internal/tui/features.go:260`

## Fix
Add a constant at the top of the file:

```go
const maxDoneItemsShown = 5
```

Replace both occurrences of the magic number with this constant.