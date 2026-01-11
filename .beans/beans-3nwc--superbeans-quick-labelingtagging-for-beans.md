---
# beans-3nwc
title: 'SUPERBEANS: Quick labeling/tagging for beans'
status: scrapped
type: feature
priority: normal
tags:
    - artifact:design
    - idea
created_at: 2026-01-02T00:29:07Z
updated_at: 2026-01-02T21:49:31Z
---

Add quick tag editing UI to add/remove tags from selected bean without leaving the TUI.

## Design

### Trigger
- Press `t` on selected bean to open tag picker modal
- Works in both features view (on feature) and detail view (on any bean)

### Modal Layout
```
┌─────────────────────────────────────────────────┐
│ Edit Tags: "Custom Beans UI for SUPERBEANS"     │
│                                                 │
│ CURRENT TAGS                                    │
│ [1] artifact:impl  ×                            │
│ [2] needs-review   ×                            │
│                                                 │
│ COMMON TAGS                                     │
│ [a] artifact:research                           │
│ [b] artifact:design                             │
│ [c] artifact:plan                               │
│ [d] artifact:impl      ●                        │
│ [e] needs-review       ●                        │
│ [f] idea                                        │
│ [g] blocked                                     │
│                                                 │
│ [+] add custom tag   [enter] save   [esc] cancel│
└─────────────────────────────────────────────────┘
```

### Keyboard Shortcuts
- `1-9`: Remove existing tag by number
- `a-z`: Toggle common tag
- `+` or `n`: Open text input to add custom tag
- `enter`: Save changes
- `esc`: Cancel

### Common Tags
Automatically populated from:
1. Artifact tags: `artifact:research`, `artifact:design`, `artifact:plan`, `artifact:impl`
2. Workflow tags: `needs-review`, `blocked`, `idea`
3. Tags used elsewhere in the project (sorted by frequency)

### State
```go
type tagPickerModel struct {
    bean        *bean.Bean
    currentTags []string        // tags currently on the bean
    commonTags  []string        // suggested tags to add
    selected    map[string]bool // toggled state
    inputMode   bool            // true when typing custom tag
    inputValue  string
}
```

### Files to Create/Modify
- `superbeans/internal/tui/tagpicker.go` - new modal model
- `superbeans/internal/tui/app.go` - wire up `t` key
- Save changes via `core.Update(bean)` on apply