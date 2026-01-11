---
# beans-ipdp
title: 'Impl: Phase 4 - Features list preview panel'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T21:45:06Z
updated_at: 2026-01-05T22:24:41Z
parent: beans-rmtd
---

## Phase 4: Features list tmux preview panel

**Scope clarification:** This phase adds ONLY tmux preview to features view (not body preview - that wasn't requested and would add scope).

**Files:**
- Modify: `internal/supertui/features.go` (add tmux preview panel, v key handler)
- Modify: `internal/supertui/app.go` (handle cyclePreviewModeMsg from features)

---

### Task 4.1: Add preview state to featuresModel

**Step 1: Update featuresModel struct**

In `features.go`, add to the struct:
```go
type featuresModel struct {
	resolver *graph.Resolver
	cursor   int
	width    int
	height   int

	// Grouped features
	inProgress []FeatureItem
	todo       []FeatureItem
	done       []FeatureItem

	// Flattened for cursor navigation
	items []FeatureItem

	// Error state
	loadError error

	// Tmux preview state
	previewScroll int
	tmuxContent   string // cached tmux content for selected feature
}
```

Note: We don't store previewMode here - App owns it.

---

### Task 4.2: Handle v key and scroll in features view

**Step 1: Add key handlers to featuresModel.Update**

```go
case "v":
	m.previewScroll = 0
	return m, func() tea.Msg {
		return cyclePreviewModeMsg{}
	}
case "ctrl+d":
	m.previewScroll += 5
case "ctrl+u":
	m.previewScroll = max(0, m.previewScroll-5)
```

**Step 2: Reset scroll and clear tmuxContent on cursor change**

In j/k handlers, add:
```go
case "j", "down":
	if m.cursor < len(m.items)-1 {
		m.cursor++
		m.previewScroll = 0
		m.tmuxContent = "" // clear stale content
	}
case "k", "up":
	if m.cursor > 0 {
		m.cursor--
		m.previewScroll = 0
		m.tmuxContent = "" // clear stale content
	}
```

---

### Task 4.3: Update ViewWithStatus signature to receive previewMode

**Step 1: Update method signature**

```go
func (m featuresModel) ViewWithStatus(statusMessage string, previewMode PreviewMode) string {
```

**Step 2: Update App.View to pass previewMode**

```go
case viewFeatures:
	return a.features.ViewWithStatus(a.statusMessage, a.previewMode)
```

---

### Task 4.4: Add tmux preview rendering

**Step 1: Add previewHeight method**

```go
func (m featuresModel) previewHeight() int {
	availableHeight := m.height - 3 // footer
	return availableHeight / 2
}
```

**Step 2: Add renderTmuxPreview method**

```go
func (m featuresModel) renderTmuxPreview() (string, int) {
	selected := m.SelectedFeature()
	if selected == nil {
		return StyleMuted.Render("(no feature selected)"), 1
	}

	sessionName := GetSessionName(selected.Bean.ID)
	if sessionName == "" {
		return StyleMuted.Render("No session — press a to attach"), 1
	}

	tmuxContent := m.tmuxContent
	if tmuxContent == "" {
		return StyleMuted.Render("(loading...)"), 1
	}

	lines := strings.Split(strings.TrimRight(tmuxContent, "\n"), "\n")
	totalLines := len(lines)

	scrollPos := m.previewScroll
	if scrollPos >= totalLines {
		scrollPos = max(0, totalLines-1)
	}
	if scrollPos > 0 {
		lines = lines[scrollPos:]
	}

	contentHeight := m.previewHeight() - 1
	if len(lines) > contentHeight {
		lines = lines[:contentHeight]
	}

	return strings.Join(lines, "\n"), totalLines
}
```

---

### Task 4.5: Update ViewWithStatus to show preview panel

**Step 1: Calculate list height based on preview mode**

At the start of ViewWithStatus, add:
```go
// Calculate available height for list
listHeight := m.height - 3 // footer
previewH := 0
if previewMode == PreviewTmux {
	previewH = m.previewHeight() + 2 // +2 for divider and spacing
	listHeight -= previewH
}
```

**Step 2: Add preview panel rendering at the end**

Before the footer rendering, add:
```go
// Add tmux preview if enabled
if previewMode == PreviewTmux {
	previewContent, totalLines := m.renderTmuxPreview()
	actualPreviewH := m.previewHeight()

	// Build divider
	dividerText := " TMUX "
	if totalLines > actualPreviewH-1 {
		endLine := min(m.previewScroll+actualPreviewH-1, totalLines)
		dividerText += fmt.Sprintf("[%d-%d/%d] ", m.previewScroll+1, endLine, totalLines)
	}
	dividerPadding := (m.width - len(dividerText)) / 2
	if dividerPadding < 0 {
		dividerPadding = 0
	}
	rightPadding := m.width - dividerPadding - len(dividerText)
	if rightPadding < 0 {
		rightPadding = 0
	}
	divider := strings.Repeat("─", dividerPadding) + dividerText + strings.Repeat("─", rightPadding)

	// Pad preview to fixed height
	previewLines := strings.Split(previewContent, "\n")
	for len(previewLines) < actualPreviewH-1 {
		previewLines = append(previewLines, "")
	}
	if len(previewLines) > actualPreviewH-1 {
		previewLines = previewLines[:actualPreviewH-1]
	}

	sections = append(sections, "")
	sections = append(sections, StyleDim.Render(divider))
	sections = append(sections, strings.Join(previewLines, "\n"))
}
```

Note: PreviewBody mode is NOT handled in features view - only tmux preview.

---

### Task 4.6: Update footer to show [v] shortcut

**Step 1: Update footer shortcuts**

Change:
```go
shortcuts := "[/] search  [enter/→] drill down  [p]rops  [a]ttach  [r]efresh  [y]ank  [q]uit"
```
to:
```go
shortcuts := "[/] search  [v] tmux  [enter/→] drill down  [p]rops  [a]ttach  [r]efresh  [y]ank  [q]uit"
```

**Step 2: Run tests**

Run: `go test ./internal/supertui/ -v`
Expected: PASS

**Step 3: Commit**

```
feat(supertui): add tmux preview panel to features list view

- Add v key to toggle tmux preview in features view
- Show session pane content for selected feature
- Clear cached content on selection change to avoid stale display
- Add [v] shortcut to footer

Refs: beans-rmtd
```
