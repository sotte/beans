---
# beans-f3ig
title: 'Impl: Phase 3 - Detail view tmux preview'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T21:44:36Z
updated_at: 2026-01-05T22:15:02Z
parent: beans-rmtd
---

## Phase 3: Detail view tmux preview rendering

**Files:**
- Modify: `internal/supertui/detail.go` (add tmux preview rendering)

---

### Task 3.1: Add tmuxContent field to detailModel

**Step 1: Add field to detailModel**

In `detail.go`, add to the struct:
```go
type detailModel struct {
	feature       FeatureItem
	cursor        int
	width         int
	height        int
	previewScroll int
	tmuxContent   string // cached tmux pane content
}
```

---

### Task 3.2: Add renderTmuxPreview method

**Step 1: Add method to detail.go**

```go
// renderTmuxPreview renders the tmux pane content for the current feature's session
// Returns the rendered content and total line count for scroll bounds
func (m detailModel) renderTmuxPreview() (content string, totalLines int) {
	// Check if there's a session for this feature
	sessionName := GetSessionName(m.feature.Bean.ID)
	if sessionName == "" {
		hint := StyleMuted.Render("No session — press a to attach")
		return hint, 1
	}

	// Use cached content if available
	tmuxContent := m.tmuxContent
	if tmuxContent == "" {
		tmuxContent = StyleMuted.Render("(loading...)")
	}

	// Split into lines for scrolling
	lines := strings.Split(strings.TrimRight(tmuxContent, "\n"), "\n")
	totalLines = len(lines)

	// Apply scroll with bounds checking
	scrollPos := m.previewScroll
	if scrollPos >= totalLines {
		scrollPos = max(0, totalLines-1)
	}
	if scrollPos > 0 {
		lines = lines[scrollPos:]
	}

	// Limit to preview height
	contentHeight := m.previewHeight() - 1
	if contentHeight < 1 {
		contentHeight = 1
	}
	if len(lines) > contentHeight {
		lines = lines[:contentHeight]
	}

	return strings.Join(lines, "\n"), totalLines
}
```

---

### Task 3.3: Update ViewWithStatus to handle PreviewTmux mode

**Step 1: Update the preview rendering section**

Replace the existing preview section in `ViewWithStatus` (the `if m.showPreview {` block) with:

```go
// Add preview pane if enabled
if previewMode.ShowsPanel() {
	var previewContent string
	var totalLines int
	var dividerLabel string

	switch previewMode {
	case PreviewBody:
		previewContent, totalLines = m.renderPreview()
		dividerLabel = "PREVIEW"
	case PreviewTmux:
		previewContent, totalLines = m.renderTmuxPreview()
		dividerLabel = "TMUX"
	}

	actualPreviewH := m.previewHeight()

	// Build divider with scroll indicator
	dividerText := fmt.Sprintf(" %s ", dividerLabel)
	scrollIndicator := ""
	if totalLines > actualPreviewH-1 {
		endLine := min(m.previewScroll+actualPreviewH-1, totalLines)
		scrollIndicator = fmt.Sprintf(" [%d-%d/%d] ", m.previewScroll+1, endLine, totalLines)
	}
	dividerText += scrollIndicator

	// Calculate padding for centered divider
	dividerPadding := (m.width - len(dividerText)) / 2
	if dividerPadding < 0 {
		dividerPadding = 0
	}
	rightPadding := m.width - dividerPadding - len(dividerText)
	if rightPadding < 0 {
		rightPadding = 0
	}
	divider := strings.Repeat("─", dividerPadding) + dividerText + strings.Repeat("─", rightPadding)
	previewDivider := StyleDim.Render(divider)

	// Pad preview content to fixed height
	previewLines := strings.Split(previewContent, "\n")
	previewContentHeight := actualPreviewH - 1
	for len(previewLines) < previewContentHeight {
		previewLines = append(previewLines, "")
	}
	if len(previewLines) > previewContentHeight {
		previewLines = previewLines[:previewContentHeight]
	}
	previewContent = strings.Join(previewLines, "\n")

	content = content + "\n\n" + previewDivider + "\n" + previewContent
}
```

**Step 2: Update previewHeight calculation to use previewMode**

The existing `previewH` calculation uses `m.showPreview`. Update to use the passed `previewMode`:

```go
previewH := 0
if previewMode.ShowsPanel() {
	previewH = m.previewHeight() + 2
}
```

**Step 3: Run all tests**

Run: `go test ./internal/supertui/ -v`
Expected: PASS

**Step 4: Commit**

```
feat(supertui): add tmux preview rendering in detail view

- Add renderTmuxPreview method for displaying pane content
- Show "No session — press a to attach" hint when no session exists
- Switch between PREVIEW and TMUX divider labels based on mode

Refs: beans-rmtd
```
