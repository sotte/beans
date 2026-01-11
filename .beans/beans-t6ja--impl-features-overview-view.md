---
# beans-t6ja
title: 'Impl 3: Features Overview View'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T18:57:50Z
updated_at: 2026-01-01T19:47:48Z
parent: beans-kwuv
---

# Phase 3: Features Overview View

**Files:**
- Create: `superbeans/internal/tui/features.go`
- Create: `superbeans/internal/tui/features_test.go`
- Modify: `superbeans/internal/tui/app.go`

**Reference:** Design in [[beans-1uxh]] View 1: Features Overview

---

## Design Decision: Simple Cursor vs bubbles/list

We use a simple `cursor int` instead of `bubbles/list.Model` because:

1. **Grouped sections with headers**: Our UI has "IN PROGRESS", "TODO", "DONE" section headers that aren't selectable. bubbles/list expects a flat list - making headers work requires hacky workarounds (fake items, skip logic in delegate).

2. **Custom rendering anyway**: We're doing fully custom grouped rendering in View(). bubbles/list's rendering wouldn't help.

3. **Disabled features**: We'd disable filtering, help, and status bar - using bubbles/list just as a cursor holder.

4. **Small list**: Features list is typically 10-20 items. No need for pagination/scrolling complexity.

5. **Simplicity**: 4 lines of j/k handling vs. fighting bubbles/list's assumptions.

**When to reconsider**: If we need scrolling for long lists, built-in `/` filtering, or switch to a flat layout, bubbles/list would make sense.

---

## Step 1: Create features model with data loading

Create `superbeans/internal/tui/features.go`:

```go
package tui

import (
	"context"
	"sort"

	"github.com/anthropics/beans/internal/bean"
	"github.com/anthropics/beans/internal/graph"
	"github.com/anthropics/beans/internal/model"
	tea "github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"
)

// featuresLoadedMsg is sent when features are loaded from the resolver
type featuresLoadedMsg struct {
	inProgress []FeatureItem
	todo       []FeatureItem
	done       []FeatureItem
	err        error
}

type featuresModel struct {
	resolver *graph.Resolver
	cursor   int // Simple cursor - see design decision above
	width    int
	height   int

	// Grouped features
	inProgress []FeatureItem
	todo       []FeatureItem
	done       []FeatureItem

	// Flattened for cursor navigation (matches display order)
	items []FeatureItem

	// Error state
	loadError error
}

func newFeaturesModel(resolver *graph.Resolver, width, height int) featuresModel {
	return featuresModel{
		resolver: resolver,
		cursor:   0,
		width:    width,
		height:   height,
	}
}

func (m featuresModel) Init() tea.Cmd {
	return m.loadFeatures
}

func (m featuresModel) loadFeatures() tea.Msg {
	ctx := context.Background()

	// Query all features (type=feature, exclude draft/scrapped)
	filter := &model.BeanFilter{
		Type:          []string{"feature"},
		ExcludeStatus: []string{"draft", "scrapped"},
	}
	features, err := m.resolver.Query().Beans(ctx, filter)
	if err != nil {
		return featuresLoadedMsg{err: err}
	}

	// Build parent→children map for all beans
	allBeans, err := m.resolver.Query().Beans(ctx, nil)
	if err != nil {
		return featuresLoadedMsg{err: err}
	}
	childrenMap := make(map[string][]*bean.Bean)
	for _, b := range allBeans {
		if b.Parent != "" {
			childrenMap[b.Parent] = append(childrenMap[b.Parent], b)
		}
	}

	// Detect active tmux sessions
	sessions := DetectSessions()

	// Build feature items with computed state
	var inProgress, todo, done []FeatureItem
	for _, f := range features {
		children := childrenMap[f.ID]
		item := NewFeatureItem(f, children, sessions)

		switch f.Status {
		case "in-progress":
			inProgress = append(inProgress, item)
		case "todo":
			todo = append(todo, item)
		case "completed":
			done = append(done, item)
		}
	}

	// Sort each group by priority, then title
	sortItems := func(items []FeatureItem) {
		sort.Slice(items, func(i, j int) bool {
			pi := priorityOrder(items[i].Bean.Priority)
			pj := priorityOrder(items[j].Bean.Priority)
			if pi != pj {
				return pi < pj
			}
			return items[i].Bean.Title < items[j].Bean.Title
		})
	}
	sortItems(inProgress)
	sortItems(todo)
	sortItems(done)

	return featuresLoadedMsg{
		inProgress: inProgress,
		todo:       todo,
		done:       done,
	}
}

func priorityOrder(p string) int {
	switch p {
	case "critical":
		return 0
	case "high":
		return 1
	case "normal", "":
		return 2
	case "low":
		return 3
	case "deferred":
		return 4
	default:
		return 2
	}
}

func (m featuresModel) Update(msg tea.Msg) (featuresModel, tea.Cmd) {
	switch msg := msg.(type) {
	case featuresLoadedMsg:
		if msg.err != nil {
			m.loadError = msg.err
			return m, nil
		}
		m.loadError = nil
		m.inProgress = msg.inProgress
		m.todo = msg.todo
		m.done = msg.done
		m.rebuildItems()
		// Reset cursor if out of bounds
		if m.cursor >= len(m.items) {
			m.cursor = max(0, len(m.items)-1)
		}
		return m, nil
	case tea.KeyMsg:
		switch msg.String() {
		case "j", "down":
			if m.cursor < len(m.items)-1 {
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

func (m *featuresModel) rebuildItems() {
	m.items = nil
	m.items = append(m.items, m.inProgress...)
	m.items = append(m.items, m.todo...)
	// Limit done to most recent 5
	if len(m.done) > 5 {
		m.items = append(m.items, m.done[:5]...)
	} else {
		m.items = append(m.items, m.done...)
	}
}

func (m *featuresModel) updateSessions(sessions map[string]bool) {
	for i := range m.inProgress {
		m.inProgress[i].HasSession = sessions[m.inProgress[i].Bean.ID]
	}
	for i := range m.todo {
		m.todo[i].HasSession = sessions[m.todo[i].Bean.ID]
	}
	for i := range m.done {
		m.done[i].HasSession = sessions[m.done[i].Bean.ID]
	}
	// Also update flattened items
	for i := range m.items {
		m.items[i].HasSession = sessions[m.items[i].Bean.ID]
	}
}

func (m featuresModel) SelectedFeature() *FeatureItem {
	if m.cursor >= 0 && m.cursor < len(m.items) {
		return &m.items[m.cursor]
	}
	return nil
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

---

## Step 2: Add View method with grouped rendering

Add to `superbeans/internal/tui/features.go`:

```go
func (m featuresModel) View() string {
	var sections []string

	// Header
	header := lipgloss.NewStyle().
		Bold(true).
		Foreground(lipgloss.Color("#7C3AED")).
		Render("SUPERBEANS")
	sections = append(sections, header)
	sections = append(sections, "")

	// Error state
	if m.loadError != nil {
		errMsg := lipgloss.NewStyle().
			Foreground(lipgloss.Color("#EF4444")).
			Render("Error loading features: " + m.loadError.Error())
		sections = append(sections, errMsg)
		sections = append(sections, "")
		sections = append(sections, "Press 'r' to retry")
		return lipgloss.JoinVertical(lipgloss.Left, sections...)
	}

	// Empty state
	if len(m.items) == 0 {
		sections = append(sections, "No features found.")
		sections = append(sections, "")
		sections = append(sections, "Create a feature with: beans create \"Title\" -t feature")
		return lipgloss.JoinVertical(lipgloss.Left, sections...)
	}

	// In Progress section
	if len(m.inProgress) > 0 {
		sections = append(sections, renderSectionHeader("IN PROGRESS"))
		for i, item := range m.inProgress {
			selected := m.cursor == i
			sections = append(sections, m.renderFeatureRow(item, selected))
		}
		sections = append(sections, "")
	}

	// Todo section
	if len(m.todo) > 0 {
		offset := len(m.inProgress)
		sections = append(sections, renderSectionHeader("TODO"))
		for i, item := range m.todo {
			selected := m.cursor == (offset + i)
			sections = append(sections, m.renderFeatureRow(item, selected))
		}
		sections = append(sections, "")
	}

	// Done section (recent)
	doneItems := m.done
	if len(doneItems) > 5 {
		doneItems = doneItems[:5]
	}
	if len(doneItems) > 0 {
		offset := len(m.inProgress) + len(m.todo)
		sections = append(sections, renderSectionHeader("DONE (recent)"))
		for i, item := range doneItems {
			selected := m.cursor == (offset + i)
			sections = append(sections, m.renderFeatureRow(item, selected))
		}
	}

	// Footer
	sections = append(sections, "")
	footer := lipgloss.NewStyle().
		Foreground(lipgloss.Color("#9CA3AF")).
		Render("[enter] drill down  [a]ttach  [r]efresh  [q]uit")
	sections = append(sections, footer)

	return lipgloss.JoinVertical(lipgloss.Left, sections...)
}

func renderSectionHeader(title string) string {
	return lipgloss.NewStyle().
		Bold(true).
		Foreground(lipgloss.Color("#9CA3AF")).
		Render(title)
}

func (m featuresModel) renderFeatureRow(item FeatureItem, selected bool) string {
	// Status indicator
	var statusSymbol string
	switch item.Bean.Status {
	case "in-progress":
		statusSymbol = "●"
	case "todo":
		statusSymbol = "○"
	case "completed":
		statusSymbol = "✓"
	default:
		statusSymbol = "○"
	}

	// Review flag
	reviewFlag := "  "
	if item.NeedsReview {
		reviewFlag = "⚑ "
	}

	// Phase display
	phaseStr := item.Phase.String()

	// Session indicator
	sessionStr := "○ none"
	if item.HasSession {
		sessionStr = "● live"
	}

	// Build row
	row := lipgloss.JoinHorizontal(lipgloss.Left,
		lipgloss.NewStyle().Width(3).Render(statusSymbol),
		lipgloss.NewStyle().Width(2).Render(reviewFlag),
		lipgloss.NewStyle().Width(30).Render(truncate(item.Bean.Title, 28)),
		lipgloss.NewStyle().Width(15).Render(phaseStr),
		lipgloss.NewStyle().Width(8).Render(sessionStr),
		lipgloss.NewStyle().Foreground(lipgloss.Color("#9CA3AF")).Render(item.Bean.ID),
	)

	if selected {
		return lipgloss.NewStyle().
			Background(lipgloss.Color("#3B3B3B")).
			Render(row)
	}
	return row
}

func truncate(s string, max int) string {
	if len(s) <= max {
		return s
	}
	return s[:max-1] + "…"
}
```

---

## Step 3: Update App to use features model

Modify `superbeans/internal/tui/app.go`:

```go
package tui

import (
	"github.com/anthropics/beans/internal/beancore"
	"github.com/anthropics/beans/internal/config"
	"github.com/anthropics/beans/internal/graph"
	tea "github.com/charmbracelet/bubbletea"
)

type viewState int

const (
	viewFeatures viewState = iota
	viewDetail
)

// selectFeatureMsg is sent when user presses enter on a feature
type selectFeatureMsg struct {
	feature FeatureItem
}

type App struct {
	state    viewState
	features featuresModel
	resolver *graph.Resolver
	config   *config.Config
	width    int
	height   int
}

func New(core *beancore.Core, cfg *config.Config) *App {
	resolver := &graph.Resolver{Core: core}
	return &App{
		state:    viewFeatures,
		resolver: resolver,
		config:   cfg,
	}
}

func (a *App) Init() tea.Cmd {
	a.features = newFeaturesModel(a.resolver, a.width, a.height)
	return a.features.Init()
}

func (a *App) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		case "q", "ctrl+c":
			return a, tea.Quit
		case "enter":
			if a.state == viewFeatures {
				if f := a.features.SelectedFeature(); f != nil {
					return a, func() tea.Msg {
						return selectFeatureMsg{feature: *f}
					}
				}
			}
		}
	case tea.WindowSizeMsg:
		a.width = msg.Width
		a.height = msg.Height
		a.features.width = msg.Width
		a.features.height = msg.Height
	case selectFeatureMsg:
		// Will be handled in Phase 4
		a.state = viewDetail
		return a, nil
	}

	// Route to active view
	switch a.state {
	case viewFeatures:
		var cmd tea.Cmd
		a.features, cmd = a.features.Update(msg)
		return a, cmd
	}

	return a, nil
}

func (a *App) View() string {
	switch a.state {
	case viewFeatures:
		return a.features.View()
	case viewDetail:
		return "Detail view (Phase 4)"
	}
	return ""
}

func Run(core *beancore.Core, cfg *config.Config) error {
	app := New(core, cfg)
	p := tea.NewProgram(app, tea.WithAltScreen())
	_, err := p.Run()
	return err
}
```

---

## Step 4: Test the features view

```bash
go build -o ./superbeans-bin ./superbeans && ./superbeans-bin
```

Expected: TUI shows features grouped by status with phase and session columns. Navigation works with j/k.

---

## Step 5: Commit

```bash
git add superbeans/internal/tui/features.go superbeans/internal/tui/app.go
git commit -m "feat(superbeans): add Features Overview view

- Add featuresModel with grouped display (in-progress, todo, done)
- Load features via GraphQL with children for phase inference
- Render rows with status, review flag, phase, session, and ID
- Wire up to App with keyboard navigation

Refs: beans-t6ja"
```

---

## Checklist

- [ ] features.go created with model and loading
- [ ] View renders grouped sections
- [ ] Row rendering with all columns
- [ ] app.go updated to use features model
- [ ] Navigation works (j/k, enter)
- [ ] Compiles and runs
- [ ] Committed