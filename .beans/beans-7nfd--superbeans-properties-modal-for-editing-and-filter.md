---
# beans-7nfd
title: 'SUPERBEANS: Properties modal for editing and filtering'
status: completed
type: feature
priority: high
tags:
    - needs-review
created_at: 2026-01-02T21:49:43Z
updated_at: 2026-01-02T23:00:27Z
---

Unified modal for editing bean properties and filtering the features list.

Consolidates: beans-8rr2, beans-3nwc, beans-8xze.

## Overview

A single modal component that serves two purposes:
1. **Edit mode (`p` key)**: Edit properties of the selected bean
2. **Filter mode (`f` key)**: Filter the features list

Both modes share the same UI layout and shortcuts, differing only in behavior:
- Edit mode: single-select (radio buttons), changes one bean
- Filter mode: multi-select (checkboxes), filters the list

## Modal Layout

```
┌─ Properties ───────────────────────────────────┐
│                                                │
│ STATUS        PRIORITY       ARTIFACT          │
│ [i] in-prog●  [1] critical   [R] research      │
│ [t] todo      [2] high       [D] design        │
│ [d] draft     [3] normal●    [P] plan          │
│ [c] complete  [4] low        [I] impl●         │
│ [x] scrapped  [5] deferred                     │
│                                                │
│ [?] needs-review       [enter] apply  [esc]    │
└────────────────────────────────────────────────┘
```

Filter mode adds `[0] reset all` in the footer.

## Keyboard Shortcuts

### Status (lowercase letters)
| Key | Status |
|-----|--------|
| `i` | in-progress |
| `t` | todo |
| `d` | draft |
| `c` | completed |
| `x` | scrapped |

### Priority (numbers = P1-P5)
| Key | Priority |
|-----|----------|
| `1` | critical |
| `2` | high |
| `3` | normal |
| `4` | low |
| `5` | deferred |

### Artifact (uppercase letters)
| Key | Artifact Tag |
|-----|--------------|
| `R` | artifact:research |
| `D` | artifact:design |
| `P` | artifact:plan |
| `I` | artifact:impl |

### Flags
| Key | Tag |
|-----|-----|
| `?` | needs-review |

### Actions
| Key | Action |
|-----|--------|
| `enter` | Apply changes/filters |
| `esc` | Cancel |
| `0` | Reset all (filter mode only) |

## Visual Feedback

- Selected items show `●` indicator
- Selected labels are bold and colored
- Unselected items are dimmed gray
- Current values are pre-selected when modal opens

## Behavior Differences

| Aspect | Edit Mode (`p`) | Filter Mode (`f`) |
|--------|-----------------|-------------------|
| Trigger | `p` on any bean | `f` in features view |
| Selection | Single per category | Multiple per category |
| Pressing key | Replaces selection | Toggles selection |
| Apply | Updates bean via GraphQL | Filters feature list |
| Empty selection | Not allowed (keeps current) | Shows all (no filter) |

## Reference Implementation

See `tui-filtering` branch, `internal/tui/filterpicker.go` for modal patterns:
- Column-based layout with `lipgloss.JoinHorizontal`
- Rounded border with primary color
- Key rendering with `[x]` style
- Toggle state with `●` indicator

## Implementation

### Files to Create
- `superbeans/internal/tui/proppicker.go` - modal model (shared for both modes)

### Files to Modify
- `superbeans/internal/tui/app.go` - wire up `p` and `f` keys
- `superbeans/internal/tui/features.go` - apply filter state to feature list
- `superbeans/internal/tui/detail.go` - wire up `p` key for child beans

### State

```go
type propPickerMode int

const (
    PropPickerEdit propPickerMode = iota
    PropPickerFilter
)

type propPickerModel struct {
    mode     propPickerMode
    beanID   string          // only for edit mode

    // Current selections
    status   string          // edit: single value, filter: unused
    statuses map[string]bool // filter: multiple values, edit: unused
    priority string
    priorities map[string]bool
    artifact string
    artifacts map[string]bool
    needsReview bool

    width, height int
}
```

## Tasks

- [ ] Create `proppicker.go` with shared modal component
- [ ] Implement edit mode behavior
- [ ] Implement filter mode behavior
- [ ] Wire up `p` key in features view and detail view
- [ ] Wire up `f` key in features view
- [ ] Add filter state to features model
- [ ] Apply filters when rendering feature list
- [ ] Show active filter indicator in features view footer
