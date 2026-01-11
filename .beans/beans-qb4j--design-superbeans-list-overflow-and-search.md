---
# beans-qb4j
title: 'Design: Superbeans list overflow and search'
status: completed
type: task
priority: normal
created_at: 2026-01-05T15:35:22Z
updated_at: 2026-01-05T15:43:33Z
parent: beans-iuaq
---

Design document for handling large TODO/DONE lists in superbeans TUI, including dedicated search view.

## Problem Statement

The superbeans TUI currently shows:
- **IN PROGRESS**: All items (no limit)
- **TODO**: All items (no limit)
- **DONE**: Limited to 5 most recent

When lists grow large, this causes:
1. **Screen real estate** - sections push each other off-screen
2. **Discoverability** - hard to find specific items
3. **Visual clutter** - overwhelming when many items
4. **Navigation** - no quick way to jump to specific features

## Chosen Solution: Two-Part Approach

### Part 1: Section Limits in Main View

Each section shows a limited number of items with overflow indicator:

```
SUPERBEANS                                    [/] search
────────────────────────────────────────────────────

IN PROGRESS (3)
▸ ● Feature A                    planning    ● work  beans-1234
  ● Feature B                    building    ○ none  beans-5678
  ● Feature C                    testing     ? ask   beans-9abc

TODO (showing 5 of 11)
  ○ Feature D                                        beans-def0
  ○ Feature E                                        beans-1111
  ○ Feature F                                        beans-2222
  ○ Feature G                                        beans-3333
  ○ Feature H                                        beans-4444
  ↓ 6 more...

DONE (showing 3 of 17)
  ✓ Feature I                                        beans-5555
  ✓ Feature J                                        beans-6666
  ✓ Feature K                                        beans-7777
  ↓ 14 more...
```

### Part 2: Dedicated Search View

Press `/` to open a full-screen search across all features:

```
SEARCH                                            [esc] cancel
────────────────────────────────────────────────────

Search: auth█

3 matches:

  ● Auth token refresh           in-progress         beans-1234
  ○ Add auth logging             todo                beans-def0
  ✓ Basic auth implementation    completed           beans-3333

────────────────────────────────────────────────────
[enter] go to feature  [esc] back
```

**Key behaviors:**
- Live filtering as you type (fuzzy match on title/ID)
- Uses bubbles/list component for built-in filtering
- Results show status indicator and full bean info
- Enter navigates to selected feature
- Esc returns to main view

## Alternative Considered: Three List Views

We discussed using three separate `bubbles/list` components, one per section.

**Pros:**
- Free filtering per section
- Free pagination via bubbles
- Consistent with beans TUI

**Cons:**
- Focus management complexity (tab between sections)
- Height allocation between sections unclear
- Global filter harder (would only filter active section)
- Visual overhead of three bordered boxes

**Decision:** Parked for now. The dedicated search view provides global search without the complexity. May revisit if per-section filtering becomes important.

## Implementation Notes

- Search view can use `bubbles/list` with `SetFilteringEnabled(true)`
- Main view keeps custom rendering but adds section limits
- Section limits could be configurable or calculated based on screen height
- Consider: "+N more" could be selectable to expand section temporarily

## Open Questions

- [ ] What should the default limits be per section?
- [ ] Should limits be fixed or calculated from available height?
- [ ] Should search include non-feature beans (tasks, bugs)?