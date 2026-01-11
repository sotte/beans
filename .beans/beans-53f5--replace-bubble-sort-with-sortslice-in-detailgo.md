---
# beans-53f5
title: Replace bubble sort with sort.Slice in detail.go
status: completed
type: task
priority: high
tags:
    - artifact:impl
created_at: 2026-01-01T22:00:26Z
updated_at: 2026-01-01T22:04:23Z
parent: beans-kwuv
---

## Problem

`sortChildrenByArtifact` uses a manual bubble sort implementation instead of the standard library's `sort.Slice`.

## File
`superbeans/internal/tui/detail.go:306-344`

## Why It Matters
- O(n²) complexity vs O(n log n) for sort.Slice
- Harder to read than idiomatic Go
- Inconsistent with `featuresModel` which correctly uses `sort.Slice` (`features.go:99-108`)

## Fix
Replace the manual bubble sort with:

```go
func sortChildrenByArtifact(children []*bean.Bean) []*bean.Bean {
    sorted := make([]*bean.Bean, len(children))
    copy(sorted, children)
    
    sort.Slice(sorted, func(i, j int) bool {
        orderI := artifactOrder(sorted[i])
        orderJ := artifactOrder(sorted[j])
        if orderI != orderJ {
            return orderI < orderJ
        }
        // Secondary sort by title for stability
        return sorted[i].Title < sorted[j].Title
    })
    
    return sorted
}
```

Keep the existing `artifactOrder` helper function.