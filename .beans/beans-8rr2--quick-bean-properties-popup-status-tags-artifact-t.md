---
# beans-8rr2
title: Quick bean properties popup (status, tags, artifact type)
status: scrapped
type: feature
priority: normal
created_at: 2026-01-01T21:36:15Z
updated_at: 2026-01-02T21:49:25Z
---

## Overview

Add a popup/modal UI for quickly editing bean properties without leaving superbeans.

## Trigger

- **Shortcut**: `p` (for properties) or `space` (familiar from many TUIs)
- Works in both features overview and detail view
- Opens a modal overlay on the current screen

## Properties to Edit

1. **Status** - todo, in-progress, completed, scrapped
2. **Priority** - critical, high, normal, low, deferred  
3. **Artifact type** - research, design, plan, impl (via tag)
4. **Tags** - including needs-review, idea, etc.

## UI Suggestions

### Option A: Tabbed sections
```
┌─ Properties: beans-abc1 ─────────────────┐
│ [Status] [Priority] [Tags]               │
│                                          │
│  ○ todo                                  │
│  ● in-progress  ← current                │
│  ○ completed                             │
│  ○ scrapped                              │
│                                          │
│ [enter] select  [tab] next  [esc] close  │
└──────────────────────────────────────────┘
```

### Option B: Single list with categories (recommended)
```
┌─ Properties: beans-abc1 ─────────────────┐
│ STATUS                                   │
│  ○ todo  ● in-progress  ○ completed      │
│ PRIORITY                                 │
│  ○ high  ● normal  ○ low                 │
│ TAGS                                     │
│  ☑ artifact:impl  ☐ needs-review         │
│                                          │
│ [enter] toggle  [esc] close              │
└──────────────────────────────────────────┘
```

### Option C: Command palette style
```
> set status in-progress
  set priority high
  add tag needs-review
  remove tag artifact:impl
```

## Recommendation

**Option B (single list with categories)** is the best choice:
- Shows everything at once - quick to scan
- No tab switching needed
- Maps well to filtering use case (same UI, different action)
- Familiar checkbox/radio pattern

## Reuse for Filtering

The same component could power a filter popup:
- `/` to open filter mode
- Select statuses to show/hide
- Select types to filter
- Free-text search

## Prior Art

There is a similar filtering implementation in the `tui-filtering` branch that could be referenced or adapted.

## Implementation Notes

- Use bubbles/overlay or custom modal rendering
- Consider bubbles/list for selection
- Save changes immediately on selection (no confirm needed)
- Show current values clearly (●/☑ for selected)
