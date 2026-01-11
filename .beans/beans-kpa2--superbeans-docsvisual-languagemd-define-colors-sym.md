---
# beans-kpa2
title: 'SUPERBEANS: docs/visual-language.md - define colors, symbols and visual elements'
status: completed
type: feature
priority: normal
created_at: 2026-01-02T19:13:53Z
updated_at: 2026-01-02T19:39:24Z
---

## Goal

Create a visual language specification document that defines how colors, symbols, and other visual elements are used throughout superbeans. This ensures consistency and makes the TUI intuitive.

## Document: `docs/visual-language.md`

Should cover:

### Colors

- **Status colors**: What color represents each status (in-progress, todo, draft, completed, scrapped)
- **Type colors**: What color represents each bean type (milestone, epic, feature, task, bug)
- **Priority colors**: Visual urgency indicators (critical=red, high=yellow, etc.)
- **Session state colors**: Working (green), idle/waiting (amber), no session (gray)
- **Semantic colors**: Error states, warnings, success feedback

### Symbols

- **Session indicators**: `●` live, `○` none, `◐` idle (from beans-lb0j)
- **Status indicators**: Checkmarks, spinners, etc.
- **Relationship indicators**: Parent/child, blocking/blocked-by
- **Navigation hints**: Keyboard shortcuts, selection states

### Layout Principles

- Information hierarchy (what's most important)
- Density vs readability tradeoffs
- Responsive behavior for different terminal sizes

### Accessibility

- Color-blind friendly palette considerations
- Sufficient contrast ratios
- Symbol meanings don't rely solely on color

## Tasks

- [x] Audit current color/symbol usage in codebase
- [x] Document existing conventions
- [x] Identify inconsistencies
- [x] Define the canonical visual language
- [x] Write `docs/visual-language.md`

## Summary of Changes

Created `docs/visual-language.md` documenting:

- **Base color palette**: 9 semantic colors with hex values and usage
- **Status colors**: in-progress (yellow), todo (green), draft (blue), completed/scrapped (gray)
- **Type colors**: milestone (cyan), epic (purple), bug (red), feature (green), task (blue)
- **Priority colors and symbols**: critical (‼ red), high (! amber), normal (none), low (↓ gray), deferred (→ gray)
- **Symbols**: blocking indicators (●/○), selection cursors (▌/▸), tree connectors (├─/└─)
- **Short codes**: single-letter codes for types (M/E/B/F/T) and statuses (D/T/I/C/S)
- **Layout principles**: column widths, responsive thresholds, tag display rules
- **Accessibility**: color-blind considerations, symbol redundancy, contrast notes
- **Configuration**: color name mappings for user customization

Note: Session state colors (working/idle/no session) are mentioned in the bean spec but not yet implemented in the codebase - this is related to beans-lb0j.