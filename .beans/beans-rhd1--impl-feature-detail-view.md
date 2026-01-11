---
# beans-rhd1
title: 'Impl 4: Feature Detail View'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T18:57:50Z
updated_at: 2026-01-01T19:52:07Z
parent: beans-kwuv
---

# Phase 4: Feature Detail View

**Files:**
- Create: `superbeans/internal/tui/detail.go`
- Modify: `superbeans/internal/tui/app.go`

**Reference:** Design in [[beans-1uxh]] View 2: Feature Detail

---

## Step 1: Create detail model with pipeline summary

Create `superbeans/internal/tui/detail.go`:

```go
package tui

import (
	"fmt"
	"strings"

	"github.com/anthropics/beans/internal/bean"
	tea "github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"
)

type detailModel struct {
	feature     FeatureItem
	cursor      int
	width       int
	height      int
}

func newDetailModel(feature FeatureItem, width, height int) detailModel {
	return detailModel{
		feature: feature,
		cursor:  0,
		width:   width,
		height:  height,
	}
}

func (m detailModel) Init() tea.Cmd {
	return nil
}

func (m detailModel) Update(msg tea.Msg) (detailModel, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		case "j", "down":
			if m.cursor < len(m.feature.Children)-1 {
				m.cursor++
			}
		case "k", "up":
			if m.cursor > 0 {
				m.cursor--
			}
		}
	}
	return m, nil
}

func (m detailModel) SelectedChild() *bean.Bean {
	if m.cursor >= 0 && m.cursor < len(m.feature.Children) {
		return m.feature.Children[m.cursor]
	}
	return nil
}
```

---

## Step 2: Add pipeline summary rendering

Add to `superbeans/internal/tui/detail.go`:

**Note:** Uses `HasTag` and `GetArtifactType` from feature.go (Phase 2).

```go
// renderPipelineSummary creates the visual pipeline: ✓ Research → ✓ Design → ◐ Plan → ○ Implement
func (m detailModel) renderPipelineSummary() string {
	phases := []struct {
		name   string
		tag    string
	}{
		{"Research", "artifact:research"},
		{"Design", "artifact:design"},
		{"Plan", "artifact:plan"},
		{"Implement", "artifact:impl"},
	}

	var parts []string
	for _, phase := range phases {
		symbol := m.getPhaseSymbol(phase.tag)
		part := fmt.Sprintf("%s %s", symbol, phase.name)

		// Add progress for impl
		if phase.tag == "artifact:impl" {
			completed, total := m.countImpl()
			if total > 0 {
				part = fmt.Sprintf("%s %s (%d/%d)", symbol, phase.name, completed, total)
			}
		}
		parts = append(parts, part)
	}

	return strings.Join(parts, " → ")
}

func (m detailModel) getPhaseSymbol(tag string) string {
	hasInProgress := false
	hasCompleted := false
	hasAny := false

	for _, child := range m.feature.Children {
		if HasTag(child, tag) { // Uses shared helper from feature.go
			hasAny = true
			if child.Status == "completed" {
				hasCompleted = true
			} else if child.Status == "in-progress" {
				hasInProgress = true
			}
		}
	}

	if !hasAny {
		return "○"
	}
	if hasInProgress {
		return "◐"
	}
	if hasCompleted {
		return "✓"
	}
	return "○"
}

func (m detailModel) countImpl() (completed, total int) {
	for _, child := range m.feature.Children {
		if HasTag(child, "artifact:impl") { // Uses shared helper
			total++
			if child.Status == "completed" {
				completed++
			}
		}
	}
	return
}
```

---

## Step 3: Add View method with children list

Add to `superbeans/internal/tui/detail.go`:

```go
func (m detailModel) View() string {
	var sections []string

	// Header with feature info
	headerStyle := lipgloss.NewStyle().Bold(true)
	header := fmt.Sprintf("← %s  %s", m.feature.Bean.ID, m.feature.Bean.Title)

	sessionInfo := "○ none"
	if m.feature.HasSession {
		sessionInfo = "● live  [a]ttach"
	}

	headerLine := lipgloss.JoinHorizontal(lipgloss.Left,
		headerStyle.Render(header),
		lipgloss.NewStyle().Foreground(lipgloss.Color("#9CA3AF")).PaddingLeft(2).Render(sessionInfo),
	)
	sections = append(sections, headerLine)
	sections = append(sections, strings.Repeat("─", min(m.width, 80)))

	// Pipeline summary
	sections = append(sections, "")
	pipeline := m.renderPipelineSummary()
	sections = append(sections, pipeline)
	sections = append(sections, "")

	// Parent feature row
	parentRow := m.renderChildRow(m.feature.Bean, -1, false, true)
	sections = append(sections, parentRow)
	sections = append(sections, strings.Repeat("─", min(m.width, 80)))

	// Children list
	for i, child := range m.feature.Children {
		selected := i == m.cursor
		row := m.renderChildRow(child, i, selected, false)
		sections = append(sections, row)
	}

	// Footer
	sections = append(sections, "")
	footer := lipgloss.NewStyle().
		Foreground(lipgloss.Color("#9CA3AF")).
		Render("[enter] view bean  [a]ttach  [esc] back  [q]uit")
	sections = append(sections, footer)

	return lipgloss.JoinVertical(lipgloss.Left, sections...)
}

func (m detailModel) renderChildRow(b *bean.Bean, index int, selected, isParent bool) string {
	// Status symbol
	var statusSymbol string
	switch b.Status {
	case "completed":
		statusSymbol = "✓"
	case "in-progress":
		statusSymbol = "●"
	default:
		statusSymbol = "○"
	}

	// Review flag - uses shared helper
	reviewFlag := "  "
	if HasTag(b, "needs-review") {
		reviewFlag = "⚑ "
	}

	// Artifact/type label - uses shared helper
	artifactLabel := GetArtifactType(b)
	if artifactLabel == "" {
		artifactLabel = b.Type
	}

	// Cursor indicator
	cursor := "  "
	if selected {
		cursor = "▌ "
	}

	// Build row
	row := lipgloss.JoinHorizontal(lipgloss.Left,
		lipgloss.NewStyle().Width(2).Render(cursor),
		lipgloss.NewStyle().Width(3).Render(statusSymbol),
		lipgloss.NewStyle().Width(2).Render(reviewFlag),
		lipgloss.NewStyle().Width(40).Render(truncate(b.Title, 38)),
		lipgloss.NewStyle().Width(12).Foreground(lipgloss.Color("#9CA3AF")).Render(artifactLabel),
		lipgloss.NewStyle().Foreground(lipgloss.Color("#9CA3AF")).Render(b.ID),
	)

	if selected {
		return lipgloss.NewStyle().
			Background(lipgloss.Color("#3B3B3B")).
			Render(row)
	}
	return row
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```

---

## Step 4: Update App to handle detail view

Modify `superbeans/internal/tui/app.go` to add detail model and navigation:

```go
// Add to App struct:
type App struct {
	state    viewState
	features featuresModel
	detail   detailModel      // Add this
	resolver *graph.Resolver
	config   *config.Config
	width    int
	height   int
}

// Update Update method:
func (a *App) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		case "q", "ctrl+c":
			return a, tea.Quit
		case "esc":
			if a.state == viewDetail {
				a.state = viewFeatures
				return a, nil
			}
		case "enter":
			if a.state == viewFeatures {
				if f := a.features.SelectedFeature(); f != nil {
					a.detail = newDetailModel(*f, a.width, a.height)
					a.state = viewDetail
					return a, a.detail.Init()
				}
			}
		}
	case tea.WindowSizeMsg:
		a.width = msg.Width
		a.height = msg.Height
		a.features.width = msg.Width
		a.features.height = msg.Height
		a.detail.width = msg.Width
		a.detail.height = msg.Height
	}

	// Route to active view
	switch a.state {
	case viewFeatures:
		var cmd tea.Cmd
		a.features, cmd = a.features.Update(msg)
		return a, cmd
	case viewDetail:
		var cmd tea.Cmd
		a.detail, cmd = a.detail.Update(msg)
		return a, cmd
	}

	return a, nil
}

// Update View method:
func (a *App) View() string {
	switch a.state {
	case viewFeatures:
		return a.features.View()
	case viewDetail:
		return a.detail.View()
	}
	return ""
}
```

---

## Step 5: Test the detail view

```bash
go build -o ./superbeans-bin ./superbeans && ./superbeans-bin
```

Expected:
- Features list shows, press enter to drill down
- Detail view shows pipeline summary and children
- esc returns to features list
- j/k navigates children

---

## Step 6: Commit

```bash
git add superbeans/internal/tui/detail.go superbeans/internal/tui/app.go
git commit -m "feat(superbeans): add Feature Detail view

- Add detailModel with pipeline summary rendering
- Show children list with status, review flag, artifact type
- Wire up navigation: enter to drill down, esc to go back
- Cursor navigation within children list

Refs: beans-rhd1"
```

---

## Checklist

- [ ] detail.go created with model
- [ ] Pipeline summary renders correctly
- [ ] Children list with all columns
- [ ] app.go updated with detail routing
- [ ] Navigation works (enter, esc, j/k)
- [ ] Compiles and runs
- [ ] Committed