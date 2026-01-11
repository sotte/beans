---
# beans-h9rf
title: Extract colors and styling into dedicated styles.go file
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T21:07:48Z
updated_at: 2026-01-01T21:26:02Z
parent: beans-kwuv
---

Created superbeans/internal/tui/styles.go with centralized color and style definitions. Refactored features.go and detail.go to use shared styles (StyleHeader, StyleMuted, StyleDim, StyleActive, StyleError, StyleStatus, StyleHighlight). Uses 256-color grayscale palette for better control: 245 for muted, 240 for dimmed, 236 for highlight background. Added CursorIndicator constant (▶) for unified row selection across views.