---
# beans-gnbj
title: 'Implementation Plan: Properties Modal (Edit Mode)'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
    - needs-review
created_at: 2026-01-02T22:00:12Z
updated_at: 2026-01-02T22:11:55Z
parent: beans-7nfd
---

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a properties modal to superbeans TUI for editing bean status, priority, and artifact tags.

**Architecture:** Modal overlay pattern adapted from main beans TUI. Single `proppicker.go` component with column-based layout. State machine in `app.go` handles modal lifecycle.

**Tech Stack:** Bubble Tea, Lipgloss, existing GraphQL mutations.

**Reference Docs:**
- `docs/properties-modal.md` - User-facing documentation
- `docs/visual-language.md` - Color and symbol conventions

---

## Phase 1: Modal Infrastructure

Copy modal utilities from main TUI and add view state for modal.

### Files

- Create: `superbeans/internal/tui/modal.go`
- Modify: `superbeans/internal/tui/app.go:18-23` (add viewState)
- Modify: `superbeans/internal/tui/app.go:48-59` (add modal to App struct)

### Code

**modal.go** - Copy overlay utilities:
```go
package tui

import (
	"strings"

	"github.com/charmbracelet/lipgloss"
)

// overlayModal places a modal on top of a background view
func overlayModal(bgView, modal string, width, height int) string {
	// Split background into lines
	bgLines := strings.Split(bgView, "\n")

	// Pad or truncate background to fill the screen
	for len(bgLines) < height {
		bgLines = append(bgLines, "")
	}
	if len(bgLines) > height {
		bgLines = bgLines[:height]
	}

	// Dim the background
	dimStyle := lipgloss.NewStyle().Foreground(lipgloss.Color("#555"))
	for i, line := range bgLines {
		bgLines[i] = dimStyle.Render(stripAnsi(line))
	}

	// Split modal into lines
	modalLines := strings.Split(modal, "\n")
	modalHeight := len(modalLines)
	modalWidth := lipgloss.Width(modal)

	// Calculate center position
	startY := (height - modalHeight) / 2
	startX := (width - modalWidth) / 2
	if startY < 0 {
		startY = 0
	}
	if startX < 0 {
		startX = 0
	}

	// Overlay modal onto background
	for i, modalLine := range modalLines {
		bgY := startY + i
		if bgY >= 0 && bgY < len(bgLines) {
			bgLines[bgY] = overlayLine(bgLines[bgY], modalLine, startX, width)
		}
	}

	return strings.Join(bgLines, "\n")
}

// overlayLine places a modal line on top of a background line at position x
func overlayLine(bgLine, modalLine string, startX, maxWidth int) string {
	bgRunes := []rune(stripAnsi(bgLine))
	for len(bgRunes) < maxWidth {
		bgRunes = append(bgRunes, ' ')
	}

	prefix := string(bgRunes[:startX])
	modalWidth := lipgloss.Width(modalLine)
	suffixStart := startX + modalWidth
	suffix := ""
	if suffixStart < len(bgRunes) {
		suffix = string(bgRunes[suffixStart:])
	}

	dimStyle := lipgloss.NewStyle().Foreground(lipgloss.Color("#555"))
	return dimStyle.Render(prefix) + modalLine + dimStyle.Render(suffix)
}

// stripAnsi removes ANSI escape codes from a string
func stripAnsi(s string) string {
	result := strings.Builder{}
	inEscape := false
	for _, r := range s {
		if r == '\x1b' {
			inEscape = true
			continue
		}
		if inEscape {
			if (r >= 'a' && r <= 'z') || (r >= 'A' && r <= 'Z') {
				inEscape = false
			}
			continue
		}
		result.WriteRune(r)
	}
	return result.String()
}
```

**app.go viewState** - Add modal state:
```go
const (
	viewFeatures viewState = iota
	viewDetail
	viewPropPicker  // Add this
)
```

**app.go App struct** - Add modal fields:
```go
type App struct {
	state         viewState
	previousState viewState    // Add: for modal return
	features      featuresModel
	detail        detailModel
	propPicker    propPickerModel  // Add: modal model
	// ... rest unchanged
}
```

### Commit

```
feat(superbeans): add modal infrastructure

- Copy overlay utilities from main TUI
- Add viewPropPicker state
- Add previousState for modal return navigation

Refs: beans-7nfd
```

---

## Phase 2: PropPicker Model

Create the properties picker component with column layout and keyboard shortcuts.

### Files

- Create: `superbeans/internal/tui/proppicker.go`

### Code

**proppicker.go**:
```go
package tui

import (
	"strings"

	"github.com/charmbracelet/lipgloss"
	tea "github.com/charmbracelet/bubbletea"
	"github.com/hmans/beans/internal/bean"
)

// Message types
type propPickerClosedMsg struct{}
type propPickerAppliedMsg struct {
	beanID   string
	status   string
	priority string
	artifact string // "research", "design", "plan", "impl", or ""
	review   bool   // needs-review flag
}

type propPickerModel struct {
	beanID   string
	beanTitle string

	// Current selections
	status   string
	priority string
	artifact string
	review   bool

	// Original values (for detecting changes)
	origStatus   string
	origPriority string
	origArtifact string
	origReview   bool

	width, height int
}

func newPropPickerModel(b *bean.Bean, width, height int) propPickerModel {
	// Extract artifact from tags
	artifact := ""
	review := false
	for _, tag := range b.Tags {
		switch tag {
		case "artifact:research":
			artifact = "research"
		case "artifact:design":
			artifact = "design"
		case "artifact:plan":
			artifact = "plan"
		case "artifact:impl":
			artifact = "impl"
		case "needs-review":
			review = true
		}
	}

	return propPickerModel{
		beanID:       b.ID,
		beanTitle:    b.Title,
		status:       b.Status,
		priority:     b.Priority,
		artifact:     artifact,
		review:       review,
		origStatus:   b.Status,
		origPriority: b.Priority,
		origArtifact: artifact,
		origReview:   review,
		width:        width,
		height:       height,
	}
}

func (m propPickerModel) Init() tea.Cmd {
	return nil
}

func (m propPickerModel) Update(msg tea.Msg) (propPickerModel, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		// Status keys (lowercase)
		case "i":
			m.status = "in-progress"
		case "t":
			m.status = "todo"
		case "d":
			m.status = "draft"
		case "c":
			m.status = "completed"
		case "x":
			m.status = "scrapped"

		// Priority keys (numbers)
		case "1":
			m.priority = "critical"
		case "2":
			m.priority = "high"
		case "3":
			m.priority = "normal"
		case "4":
			m.priority = "low"
		case "5":
			m.priority = "deferred"

		// Artifact keys (uppercase)
		case "R":
			if m.artifact == "research" {
				m.artifact = ""
			} else {
				m.artifact = "research"
			}
		case "D":
			if m.artifact == "design" {
				m.artifact = ""
			} else {
				m.artifact = "design"
			}
		case "P":
			if m.artifact == "plan" {
				m.artifact = ""
			} else {
				m.artifact = "plan"
			}
		case "I":
			if m.artifact == "impl" {
				m.artifact = ""
			} else {
				m.artifact = "impl"
			}

		// Review flag
		case "?":
			m.review = !m.review

		// Actions
		case "enter":
			return m, func() tea.Msg {
				return propPickerAppliedMsg{
					beanID:   m.beanID,
					status:   m.status,
					priority: m.priority,
					artifact: m.artifact,
					review:   m.review,
				}
			}
		case "esc":
			return m, func() tea.Msg {
				return propPickerClosedMsg{}
			}
		}

	case tea.WindowSizeMsg:
		m.width = msg.Width
		m.height = msg.Height
	}

	return m, nil
}

func (m propPickerModel) View() string {
	// Styles
	selected := lipgloss.NewStyle().Bold(true)
	dimmed := StyleDim
	keyStyle := StyleMuted

	// Indicator for selected items
	indicator := StyleActive.Render("●")

	// Build status column
	statusItems := []struct{ key, label, value string }{
		{"i", "in-prog", "in-progress"},
		{"t", "todo", "todo"},
		{"d", "draft", "draft"},
		{"c", "complete", "completed"},
		{"x", "scrapped", "scrapped"},
	}
	var statusLines []string
	statusLines = append(statusLines, selected.Render("STATUS"))
	for _, item := range statusItems {
		line := keyStyle.Render("["+item.key+"]") + " "
		if m.status == item.value {
			line += selected.Render(item.label) + indicator
		} else {
			line += dimmed.Render(item.label)
		}
		statusLines = append(statusLines, line)
	}
	statusCol := strings.Join(statusLines, "\n")

	// Build priority column
	priorityItems := []struct{ key, label, value string }{
		{"1", "critical", "critical"},
		{"2", "high", "high"},
		{"3", "normal", "normal"},
		{"4", "low", "low"},
		{"5", "deferred", "deferred"},
	}
	var priorityLines []string
	priorityLines = append(priorityLines, selected.Render("PRIORITY"))
	for _, item := range priorityItems {
		line := keyStyle.Render("["+item.key+"]") + " "
		if m.priority == item.value {
			line += selected.Render(item.label) + indicator
		} else {
			line += dimmed.Render(item.label)
		}
		priorityLines = append(priorityLines, line)
	}
	priorityCol := strings.Join(priorityLines, "\n")

	// Build artifact column
	artifactItems := []struct{ key, label, value string }{
		{"R", "research", "research"},
		{"D", "design", "design"},
		{"P", "plan", "plan"},
		{"I", "impl", "impl"},
	}
	var artifactLines []string
	artifactLines = append(artifactLines, selected.Render("ARTIFACT"))
	for _, item := range artifactItems {
		line := keyStyle.Render("["+item.key+"]") + " "
		if m.artifact == item.value {
			line += selected.Render(item.label) + indicator
		} else {
			line += dimmed.Render(item.label)
		}
		artifactLines = append(artifactLines, line)
	}
	// Pad to match other columns
	artifactLines = append(artifactLines, "")
	artifactCol := strings.Join(artifactLines, "\n")

	// Join columns horizontally with spacing
	colWidth := 14
	statusColStyled := lipgloss.NewStyle().Width(colWidth).Render(statusCol)
	priorityColStyled := lipgloss.NewStyle().Width(colWidth).Render(priorityCol)
	artifactColStyled := lipgloss.NewStyle().Width(colWidth).Render(artifactCol)

	columns := lipgloss.JoinHorizontal(lipgloss.Top,
		statusColStyled,
		priorityColStyled,
		artifactColStyled,
	)

	// Review flag row
	reviewLine := keyStyle.Render("[?]") + " "
	if m.review {
		reviewLine += selected.Render("needs-review") + indicator
	} else {
		reviewLine += dimmed.Render("needs-review")
	}

	// Footer with actions
	footer := keyStyle.Render("[enter]") + " apply  " + keyStyle.Render("[esc]") + " cancel"

	// Assemble content
	content := columns + "\n\n" + reviewLine + "\n\n" + footer

	// Modal border
	modalWidth := 48
	border := lipgloss.NewStyle().
		Border(lipgloss.RoundedBorder()).
		BorderForeground(ColorPurple).
		Padding(1, 2).
		Width(modalWidth)

	// Title
	title := lipgloss.NewStyle().Bold(true).Foreground(ColorPurple).Render("Properties")
	beanInfo := StyleMuted.Render(m.beanID)

	return border.Render(title + "\n" + beanInfo + "\n\n" + content)
}

func (m propPickerModel) ModalView(bgView string, width, height int) string {
	return overlayModal(bgView, m.View(), width, height)
}
```

### Commit

```
feat(superbeans): add properties picker modal component

- Column layout for status, priority, artifact
- Keyboard shortcuts: i/t/d/c/x for status, 1-5 for priority
- Shift+R/D/P/I for artifact tags, ? for needs-review
- Visual feedback with selected indicator

Refs: beans-7nfd
```

---

## Phase 3: Wire Up Modal in App

Connect the modal to the app state machine and handle key bindings.

### Files

- Modify: `superbeans/internal/tui/app.go`

### Changes

1. Add message type for opening modal:
```go
type openPropPickerMsg struct {
	bean *bean.Bean
}
```

2. Handle `p` key in Update to open modal:
```go
case "p":
	var b *bean.Bean
	if a.state == viewFeatures {
		if f := a.features.SelectedFeature(); f != nil {
			b = f.Bean
		}
	} else if a.state == viewDetail {
		b = a.detail.SelectedBean()
	}
	if b != nil {
		a.previousState = a.state
		a.propPicker = newPropPickerModel(b, a.width, a.height)
		a.state = viewPropPicker
		return a, a.propPicker.Init()
	}
```

3. Handle modal messages:
```go
case propPickerClosedMsg:
	a.state = a.previousState
	return a, nil

case propPickerAppliedMsg:
	a.state = a.previousState
	return a, a.applyPropertyChanges(msg)
```

4. Route updates to modal when active:
```go
case viewPropPicker:
	var cmd tea.Cmd
	a.propPicker, cmd = a.propPicker.Update(msg)
	return a, cmd
```

5. Render modal in View:
```go
case viewPropPicker:
	return a.propPicker.ModalView(a.getBackgroundView(), a.width, a.height)
```

6. Add helper methods:
```go
func (a *App) getBackgroundView() string {
	switch a.previousState {
	case viewFeatures:
		return a.features.ViewWithStatus(a.statusMessage)
	case viewDetail:
		return a.detail.ViewWithStatus(a.statusMessage)
	}
	return ""
}

func (a *App) applyPropertyChanges(msg propPickerAppliedMsg) tea.Cmd {
	return func() tea.Msg {
		// Load the bean
		b, err := a.core.Load(msg.beanID)
		if err != nil {
			return propertyUpdateFailedMsg{err: err}
		}

		// Apply changes
		b.Status = msg.status
		b.Priority = msg.priority

		// Update artifact tags
		artifactTags := []string{"artifact:research", "artifact:design", "artifact:plan", "artifact:impl"}
		for _, tag := range artifactTags {
			b.RemoveTag(tag)
		}
		if msg.artifact != "" {
			b.AddTag("artifact:" + msg.artifact)
		}

		// Update review flag
		if msg.review {
			b.AddTag("needs-review")
		} else {
			b.RemoveTag("needs-review")
		}

		// Save
		if err := a.core.Update(b); err != nil {
			return propertyUpdateFailedMsg{err: err}
		}

		return propertyUpdatedMsg{beanID: msg.beanID}
	}
}
```

7. Add result message types and handlers:
```go
type propertyUpdatedMsg struct{ beanID string }
type propertyUpdateFailedMsg struct{ err error }

// In Update switch:
case propertyUpdatedMsg:
	a.statusMessage = fmt.Sprintf("Updated %s", msg.beanID)
	return a, tea.Batch(
		a.refreshCurrentView(),
		tea.Tick(2*time.Second, func(t time.Time) tea.Msg {
			return clearStatusMsg{}
		}),
	)

case propertyUpdateFailedMsg:
	a.statusMessage = fmt.Sprintf("Failed: %v", msg.err)
	return a, tea.Tick(3*time.Second, func(t time.Time) tea.Msg {
		return clearStatusMsg{}
	})
```

8. Add refresh helper:
```go
func (a *App) refreshCurrentView() tea.Cmd {
	if a.previousState == viewDetail {
		return a.refreshDetailView
	}
	return a.features.loadFeatures
}
```

### Commit

```
feat(superbeans): wire up properties modal

- Press 'p' to open modal from features or detail view
- Apply changes via GraphQL mutation
- Refresh view after successful update
- Show status message for success/failure

Refs: beans-7nfd
```

---

## Phase 4: Update Footer Help

Add `p` key hint to the features and detail view footers.

### Files

- Modify: `superbeans/internal/tui/features.go`
- Modify: `superbeans/internal/tui/detail.go`

### Changes

**features.go** - Add to footer shortcuts (around line 294):
```go
// Current: "↑↓/jk nav • enter drill • y yank • a attach • r refresh • q quit"
// New:     "↑↓/jk nav • enter drill • p props • y yank • a attach • r refresh • q quit"
```

**detail.go** - Add to footer shortcuts (similar location):
```go
// Add "p props" to the shortcut list
```

### Commit

```
feat(superbeans): add properties shortcut to footer help

Refs: beans-7nfd
```

---

## Implementation Phases

| Phase | Bean ID | Description |
|-------|---------|-------------|
| 1 | beans-obth | Modal infrastructure (modal.go, viewState) |
| 2 | beans-m191 | PropPicker component |
| 3 | beans-gplg | Wire up modal in app |
| 4 | beans-oii7 | Update footer help |
