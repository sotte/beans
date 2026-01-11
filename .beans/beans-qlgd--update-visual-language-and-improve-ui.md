---
# beans-qlgd
title: Update visual language and improve UI
status: completed
type: feature
priority: normal
created_at: 2026-01-05T20:03:01Z
updated_at: 2026-01-05T21:18:09Z
---

Update superbeans TUI to match visual-language.md spec:
- Use ANSI colors 0-15 only (no hex, no 256-color)
- Update status symbols to Nerdfont checkboxes
- Update priority symbols to Nerdfont alerts
- Update session symbols (robot, pause, dot)
- Update phase completed symbol to filled circle

## Tasks
- [x] Update styles.go colors and symbols
- [x] Update phase.go completed symbol
- [x] Update modal.go hex colors
- [x] Update features.go legend and session display
- [x] Update detail.go session icons
- [x] Update tests
- [x] Build and verify

## Summary of Changes
- Renamed colors: ColorPurple→ColorMagenta, ColorAmber→ColorYellow, ColorGray/ColorDimGray→ColorDim
- Added symbol constants for all visual elements
- Created RenderSessionSymbol() and RenderPhaseSymbol() helper functions
- Updated all files to use new constants and ANSI-only colors
- Tests updated to use symbol constants