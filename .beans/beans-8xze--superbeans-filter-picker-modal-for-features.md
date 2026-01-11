---
# beans-8xze
title: 'SUPERBEANS: Filter picker modal for features'
status: scrapped
type: feature
priority: normal
tags:
    - artifact:design
    - idea
created_at: 2026-01-02T00:29:06Z
updated_at: 2026-01-02T21:49:36Z
---

Add filter picker modal (like tui-filtering branch) to filter features by status, priority, tags, and phase.

## Reference Implementation

See `tui-filtering` branch, specifically `internal/tui/filterpicker.go`.

## Design

### Trigger
- Press `f` in features view to open filter modal
- Modal overlays current view

### Modal Layout
```
┌─────────────────────────────────────────────────┐
│ Filter Features                                 │
│                                                 │
│ STATUS          PRIORITY        PHASE           │
│                                                 │
│ [1] draft       [q] critical    [r] research    │
│ [2] todo    ●   [w] high        [d] design      │
│ [3] in-progress [e] normal  ●   [p] plan        │
│ [4] completed   [r] low         [i] impl        │
│                 [t] deferred    [c] implemented │
│                                                 │
│ [x] reset all   [enter] apply   [esc] cancel    │
└─────────────────────────────────────────────────┘
```

### Keyboard Shortcuts
- `1-4`: Toggle status filters (draft, todo, in-progress, completed)
- `q/w/e/r/t`: Toggle priority filters
- `r/d/p/i/c`: Toggle phase filters
- `x`: Reset all filters
- `enter`: Apply and close
- `esc`: Cancel and close

### Filter State
```go
type filterState struct {
    statuses   []string  // empty = show all
    priorities []string
    phases     []string
}
```

### Visual Feedback
- Selected filters show `●` indicator
- Selected labels are bold/colored
- Unselected are dimmed gray
- Filter bar below list shows active filters when modal closed

### Files to Create/Modify
- `superbeans/internal/tui/filterpicker.go` - new modal model
- `superbeans/internal/tui/app.go` - wire up `f` key, add filter state
- `superbeans/internal/tui/features.go` - apply filters to feature list