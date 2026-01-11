---
# beans-3490
title: Show priority in superbeans
status: completed
type: feature
priority: high
created_at: 2026-01-01T23:02:51Z
updated_at: 2026-01-02T22:35:28Z
---

Display bean priority in the superbeans TUI views

## Tasks

- [x] Update `docs/visual-language.md` - add Priority column to Column Widths table
- [x] Update `superbeans/internal/tui/styles.go` - add priority styles and RenderPrioritySymbol function
- [x] Update `superbeans/internal/tui/features.go` - add priority column to renderFeatureRow()
- [x] Update `superbeans/internal/tui/detail.go` - add priority column to renderChildRow()
- [x] Build and verify

## Summary of Changes

**superbeans TUI:**
- Added priority styles: `StylePriorityCritical`, `StylePriorityHigh`, `StylePriorityLow`, `StylePriorityDeferred`
- Added `RenderPrioritySymbol()` function that returns styled symbols: ‼ (critical/red/bold), ! (high/amber/bold), ↓ (low/gray), → (deferred/gray), space (normal)
- Features overview: priority column added after cursor, before status
- Detail view: priority column added after cursor, before status for all child beans

**beans CLI TUI (bonus):**
- Also updated `internal/ui/styles.go` and `internal/tui/detail.go` for consistency

**Column order:** cursor → priority → status → review → title → phase → session → id