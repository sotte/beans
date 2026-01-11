---
# beans-8axb
title: 'Implementation Plan: Superbeans list overflow and search'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
created_at: 2026-01-05T15:40:10Z
updated_at: 2026-01-05T16:00:09Z
parent: beans-iuaq
---

# Superbeans List Overflow and Search Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Handle large TODO/DONE lists in superbeans TUI with section limits and a dedicated search view.

**Architecture:** Two-part approach: (1) Add configurable limits per section with overflow indicators in the existing `featuresModel`, (2) Create a new `searchModel` view using bubbles/list component with built-in filtering.

**Tech Stack:** Go, Bubbletea, Bubbles (list component with filtering), Lipgloss

---

## Phase 1: Section Limits in Main View

Add configurable limits to each section with overflow indicators.

### Task 1.1: Add section limit constants and configuration

**Files:**
- Modify: `superbeans/internal/tui/features.go:23-39`

**Step 1: Add section limit constants**

Add after the imports, before `featuresLoadedMsg`:

```go
// Section display limits
const (
	maxInProgressDisplay = 10  // Show all in-progress (usually few)
	maxTodoDisplay       = 5   // Limit TODO section
	maxDoneDisplay       = 3   // Limit DONE section (was hardcoded as 5)
)
```

**Step 2: Run tests to verify nothing broke**

Run: `go test ./superbeans/internal/tui/... -v`
Expected: All existing tests pass

**Step 3: Commit**

```bash
git add superbeans/internal/tui/features.go
git commit -m "feat(superbeans): add section limit constants

Refs: beans-iuaq"
```

---

### Task 1.2: Update rebuildItems to apply limits consistently

**Files:**
- Modify: `superbeans/internal/tui/features.go:183-193`
- Modify: `superbeans/internal/tui/features_test.go`

**Step 1: Write the test for section limits**

Add to `superbeans/internal/tui/features_test.go`:

```go
func TestFeaturesModel_RebuildItemsWithLimits(t *testing.T) {
	t.Run("limits todo section", func(t *testing.T) {
		m := featuresModel{}

		// Create more items than limit
		var todoItems []FeatureItem
		for i := 0; i < 10; i++ {
			todoItems = append(todoItems, FeatureItem{
				Bean: &bean.Bean{ID: fmt.Sprintf("beans-todo%02d", i), Status: "todo"},
			})
		}

		m.todo = todoItems
		m.rebuildItems()

		// items should be limited
		assert.LessOrEqual(t, len(m.items), maxTodoDisplay)
	})

	t.Run("limits done section", func(t *testing.T) {
		m := featuresModel{}

		var doneItems []FeatureItem
		for i := 0; i < 10; i++ {
			doneItems = append(doneItems, FeatureItem{
				Bean: &bean.Bean{ID: fmt.Sprintf("beans-done%02d", i), Status: "completed"},
			})
		}

		m.done = doneItems
		m.rebuildItems()

		assert.LessOrEqual(t, len(m.items), maxDoneDisplay)
	})
}
```

**Step 2: Run test to verify it fails**

Run: `go test ./superbeans/internal/tui/... -run TestFeaturesModel_RebuildItemsWithLimits -v`
Expected: FAIL (limits not applied yet)

**Step 3: Update rebuildItems function**

Replace the `rebuildItems` function:

```go
func (m *featuresModel) rebuildItems() {
	m.items = nil

	// In-progress: show all (or limit if many)
	if len(m.inProgress) > maxInProgressDisplay {
		m.items = append(m.items, m.inProgress[:maxInProgressDisplay]...)
	} else {
		m.items = append(m.items, m.inProgress...)
	}

	// Todo: apply limit
	if len(m.todo) > maxTodoDisplay {
		m.items = append(m.items, m.todo[:maxTodoDisplay]...)
	} else {
		m.items = append(m.items, m.todo...)
	}

	// Done: apply limit
	if len(m.done) > maxDoneDisplay {
		m.items = append(m.items, m.done[:maxDoneDisplay]...)
	} else {
		m.items = append(m.items, m.done...)
	}
}
```

**Step 4: Run test to verify it passes**

Run: `go test ./superbeans/internal/tui/... -run TestFeaturesModel_RebuildItemsWithLimits -v`
Expected: PASS

**Step 5: Run all tests**

Run: `go test ./superbeans/internal/tui/... -v`
Expected: All tests pass

**Step 6: Commit**

```bash
git add superbeans/internal/tui/features.go superbeans/internal/tui/features_test.go
git commit -m "feat(superbeans): apply section limits in rebuildItems

- IN PROGRESS: max 10 items
- TODO: max 5 items
- DONE: max 3 items

Refs: beans-iuaq"
```

---

### Task 1.3: Add overflow indicators to View

**Files:**
- Modify: `superbeans/internal/tui/features.go:232-340` (ViewWithStatus and helper functions)

**Step 1: Update section header helper to show counts**

Replace `renderSectionHeader` function:

```go
func renderSectionHeader(title string, shown, total int) string {
	if shown < total {
		return StyleMuted.Bold(true).Render(fmt.Sprintf("%s (showing %d of %d)", title, shown, total))
	}
	if total > 0 {
		return StyleMuted.Bold(true).Render(fmt.Sprintf("%s (%d)", title, total))
	}
	return StyleMuted.Bold(true).Render(title)
}
```

**Step 2: Add overflow indicator helper**

Add after `renderSectionHeader`:

```go
func renderOverflowIndicator(hidden int) string {
	return StyleMuted.Render(fmt.Sprintf("  ↓ %d more...", hidden))
}
```

**Step 3: Update ViewWithStatus to use new helpers**

Update the IN PROGRESS section (~line 264-274):

```go
	// In Progress section (always show, even if empty)
	inProgressShown := min(len(m.inProgress), maxInProgressDisplay)
	sections = append(sections, renderSectionHeader("IN PROGRESS", inProgressShown, len(m.inProgress)))
	if len(m.inProgress) > 0 {
		for i := 0; i < inProgressShown; i++ {
			selected := m.cursor == i
			sections = append(sections, m.renderFeatureRow(m.inProgress[i], selected))
		}
		if len(m.inProgress) > maxInProgressDisplay {
			sections = append(sections, renderOverflowIndicator(len(m.inProgress)-maxInProgressDisplay))
		}
	} else {
		sections = append(sections, StyleMuted.Render("  (none)"))
	}
	sections = append(sections, "")
```

Update the TODO section (~line 276-285):

```go
	// Todo section
	if len(m.todo) > 0 {
		offset := inProgressShown
		todoShown := min(len(m.todo), maxTodoDisplay)
		sections = append(sections, renderSectionHeader("TODO", todoShown, len(m.todo)))
		for i := 0; i < todoShown; i++ {
			selected := m.cursor == (offset + i)
			sections = append(sections, m.renderFeatureRow(m.todo[i], selected))
		}
		if len(m.todo) > maxTodoDisplay {
			sections = append(sections, renderOverflowIndicator(len(m.todo)-maxTodoDisplay))
		}
		sections = append(sections, "")
	}
```

Update the DONE section (~line 287-299):

```go
	// Done section
	if len(m.done) > 0 {
		offset := inProgressShown + min(len(m.todo), maxTodoDisplay)
		doneShown := min(len(m.done), maxDoneDisplay)
		sections = append(sections, renderSectionHeader("DONE", doneShown, len(m.done)))
		for i := 0; i < doneShown; i++ {
			selected := m.cursor == (offset + i)
			sections = append(sections, m.renderFeatureRow(m.done[i], selected))
		}
		if len(m.done) > maxDoneDisplay {
			sections = append(sections, renderOverflowIndicator(len(m.done)-maxDoneDisplay))
		}
	}
```

**Step 4: Run tests**

Run: `go test ./superbeans/internal/tui/... -v`
Expected: All tests pass

**Step 5: Manual test**

Run: `mise beans` or build and run superbeans
Expected: Sections show "(showing N of M)" when items exceed limits

**Step 6: Commit**

```bash
git add superbeans/internal/tui/features.go
git commit -m "feat(superbeans): add overflow indicators to sections

Shows '(showing N of M)' in section headers and '↓ X more...'
at the bottom of limited sections.

Refs: beans-iuaq"
```

---

## Phase 2: Dedicated Search View

Create a new search view using bubbles/list component.

### Task 2.1: Create searchModel with bubbles/list

**Files:**
- Create: `superbeans/internal/tui/search.go`
- Create: `superbeans/internal/tui/search_test.go`

**Step 1: Create search.go with basic structure**

```go
package tui

import (
	"fmt"
	"io"

	"github.com/charmbracelet/bubbles/list"
	"github.com/charmbracelet/bubbles/textinput"
	tea "github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"
)

// searchResultItem wraps a FeatureItem for the search list
type searchResultItem struct {
	feature FeatureItem
}

func (i searchResultItem) Title() string       { return i.feature.Bean.Title }
func (i searchResultItem) Description() string { return i.feature.Bean.ID }
func (i searchResultItem) FilterValue() string {
	return i.feature.Bean.Title + " " + i.feature.Bean.ID
}

// searchResultDelegate renders search result items
type searchResultDelegate struct {
	width int
}

func (d searchResultDelegate) Height() int                             { return 1 }
func (d searchResultDelegate) Spacing() int                            { return 0 }
func (d searchResultDelegate) Update(_ tea.Msg, _ *list.Model) tea.Cmd { return nil }

func (d searchResultDelegate) Render(w io.Writer, m list.Model, index int, listItem list.Item) {
	item, ok := listItem.(searchResultItem)
	if !ok {
		return
	}

	selected := index == m.Index()

	cursor := "  "
	if selected {
		cursor = CursorIndicator
	}

	// Priority symbol
	prioritySymbol := RenderPrioritySymbol(item.feature.Bean.Priority)

	// Status indicator
	var statusSymbol string
	switch item.feature.Bean.Status {
	case "in-progress":
		statusSymbol = "●"
	case "todo":
		statusSymbol = "○"
	case "completed":
		statusSymbol = "✓"
	default:
		statusSymbol = "○"
	}

	// Status label
	statusLabel := lipgloss.NewStyle().Width(12).Render(item.feature.Bean.Status)

	// Calculate title width
	titleWidth := d.width - 2 - 2 - 3 - 12 - 15 // cursor, priority, status symbol, status label, id
	if titleWidth < 20 {
		titleWidth = 20
	}

	// Build row with styles
	baseStyle := lipgloss.NewStyle()
	dimStyle := StyleMuted
	cursorStyle := lipgloss.NewStyle()
	if selected {
		baseStyle = baseStyle.Background(ColorBgHL)
		dimStyle = dimStyle.Background(ColorBgHL)
		cursorStyle = StyleCursor.Background(ColorBgHL)
	}

	rowContent := lipgloss.JoinHorizontal(lipgloss.Left,
		cursorStyle.Width(2).Render(cursor),
		baseStyle.Width(2).Render(prioritySymbol),
		baseStyle.Width(3).Render(statusSymbol),
		baseStyle.Width(12).Render(statusLabel),
		baseStyle.Width(titleWidth).Render(truncate(item.feature.Bean.Title, titleWidth-2)),
		dimStyle.Render(item.feature.Bean.ID),
	)

	if selected {
		fmt.Fprint(w, StyleHighlight.Render(rowContent))
	} else {
		fmt.Fprint(w, rowContent)
	}
}

// Message types for search
type closeSearchMsg struct{}
type selectSearchResultMsg struct {
	feature FeatureItem
}

// searchModel is the dedicated search view
type searchModel struct {
	list     list.Model
	allItems []FeatureItem
	width    int
	height   int
}

func newSearchModel(items []FeatureItem, width, height int) searchModel {
	// Convert to list items
	listItems := make([]list.Item, len(items))
	for i, item := range items {
		listItems[i] = searchResultItem{feature: item}
	}

	delegate := searchResultDelegate{width: width}
	l := list.New(listItems, delegate, width-4, height-8)
	l.Title = "Search Features"
	l.SetShowStatusBar(false)
	l.SetFilteringEnabled(true)
	l.SetShowHelp(false)
	l.SetShowPagination(true)

	// Style the list
	l.Styles.Title = StyleHeader
	l.Styles.TitleBar = lipgloss.NewStyle().Padding(0, 0, 1, 0)
	l.Styles.FilterPrompt = lipgloss.NewStyle().Foreground(ColorAmber)
	l.Styles.FilterCursor = lipgloss.NewStyle().Foreground(ColorAmber)

	return searchModel{
		list:     l,
		allItems: items,
		width:    width,
		height:   height,
	}
}

func (m searchModel) Init() tea.Cmd {
	return textinput.Blink
}

func (m searchModel) Update(msg tea.Msg) (searchModel, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.WindowSizeMsg:
		m.width = msg.Width
		m.height = msg.Height
		m.list.SetSize(msg.Width-4, msg.Height-8)
		// Update delegate width
		m.list.SetDelegate(searchResultDelegate{width: msg.Width})

	case tea.KeyMsg:
		switch msg.String() {
		case "esc":
			// If filtering, first esc clears filter; second closes search
			if m.list.FilterState() == list.Filtering {
				m.list.ResetFilter()
				return m, nil
			}
			if m.list.FilterValue() != "" {
				m.list.ResetFilter()
				return m, nil
			}
			return m, func() tea.Msg { return closeSearchMsg{} }

		case "enter":
			if item, ok := m.list.SelectedItem().(searchResultItem); ok {
				return m, func() tea.Msg {
					return selectSearchResultMsg{feature: item.feature}
				}
			}
		}
	}

	var cmd tea.Cmd
	m.list, cmd = m.list.Update(msg)
	return m, cmd
}

func (m searchModel) View() string {
	header := StyleHeader.Render("SEARCH")
	hint := StyleMuted.Render("Type to filter • enter to select • esc to close")

	border := lipgloss.NewStyle().
		Border(lipgloss.RoundedBorder()).
		BorderForeground(ColorGray).
		Width(m.width - 4).
		Padding(0, 1)

	content := lipgloss.JoinVertical(lipgloss.Left,
		header,
		hint,
		"",
		border.Render(m.list.View()),
	)

	return content
}
```

**Step 2: Create search_test.go**

```go
package tui

import (
	"testing"

	"github.com/hmans/beans/internal/bean"
	"github.com/stretchr/testify/assert"
)

func TestSearchModel_Creation(t *testing.T) {
	items := []FeatureItem{
		{Bean: &bean.Bean{ID: "beans-001", Title: "Feature One", Status: "in-progress"}},
		{Bean: &bean.Bean{ID: "beans-002", Title: "Feature Two", Status: "todo"}},
	}

	m := newSearchModel(items, 80, 24)

	assert.Equal(t, 2, len(m.allItems))
	assert.Equal(t, 80, m.width)
}

func TestSearchResultItem_FilterValue(t *testing.T) {
	item := searchResultItem{
		feature: FeatureItem{
			Bean: &bean.Bean{ID: "beans-abc", Title: "Auth Feature"},
		},
	}

	filterValue := item.FilterValue()
	assert.Contains(t, filterValue, "Auth Feature")
	assert.Contains(t, filterValue, "beans-abc")
}
```

**Step 3: Run tests**

Run: `go test ./superbeans/internal/tui/... -run TestSearch -v`
Expected: PASS

**Step 4: Commit**

```bash
git add superbeans/internal/tui/search.go superbeans/internal/tui/search_test.go
git commit -m "feat(superbeans): add dedicated search view model

Uses bubbles/list component with built-in filtering for
live search across all features.

Refs: beans-iuaq"
```

---

### Task 2.2: Integrate search view into App

**Files:**
- Modify: `superbeans/internal/tui/app.go`

**Step 1: Add viewSearch state**

Update the viewState const block (~line 20-24):

```go
const (
	viewFeatures viewState = iota
	viewDetail
	viewPropPicker
	viewSearch
)
```

**Step 2: Add search field to App struct**

Add to App struct (~line 51-64), after propPicker:

```go
	search        searchModel
```

**Step 3: Handle "/" key to open search**

In the Update function's KeyMsg switch (~line 88-162), add after the "p" case:

```go
		case "/":
			if a.state == viewFeatures {
				// Collect all features for search
				allFeatures := append(append(
					a.features.inProgress,
					a.features.todo...),
					a.features.done...,
				)
				a.search = newSearchModel(allFeatures, a.width, a.height)
				a.previousState = a.state
				a.state = viewSearch
				return a, a.search.Init()
			}
```

**Step 4: Handle search messages**

Add after the propertyUpdateFailedMsg case (~line 250-255):

```go
	case closeSearchMsg:
		a.state = a.previousState
		return a, nil

	case selectSearchResultMsg:
		a.detail = newDetailModel(msg.feature, a.width, a.height)
		a.state = viewDetail
		return a, a.detail.Init()
```

**Step 5: Route to search view**

In the view routing switch at the end of Update (~line 257-271), add:

```go
	case viewSearch:
		var cmd tea.Cmd
		a.search, cmd = a.search.Update(msg)
		return a, cmd
```

**Step 6: Render search view**

In View() (~line 276-286), add to the switch:

```go
	case viewSearch:
		return a.search.View()
```

**Step 7: Update footer to show "/" shortcut**

In `features.go`, update the shortcuts string in `renderWithFooter` (~line 307):

```go
	shortcuts := "[/] search  [enter/→] drill down  [p]rops  [a]ttach  [r]efresh  [y]ank  [q]uit"
```

**Step 8: Run tests**

Run: `go test ./superbeans/internal/tui/... -v`
Expected: All tests pass

**Step 9: Manual test**

1. Run superbeans
2. Press `/` - should open search view
3. Type to filter - results filter live
4. Press enter - should navigate to feature detail
5. Press esc - should close search

**Step 10: Commit**

```bash
git add superbeans/internal/tui/app.go superbeans/internal/tui/features.go
git commit -m "feat(superbeans): integrate search view

- Press '/' from features view to open search
- Live filtering as you type
- Enter selects and navigates to feature detail
- Esc closes search

Refs: beans-iuaq"
```

---

## Phase 3: Final Testing and Cleanup

### Task 3.1: Add integration tests

**Files:**
- Modify: `superbeans/internal/tui/app_test.go`

**Step 1: Add test for search flow**

```go
func TestApp_SearchFlow(t *testing.T) {
	t.Run("slash opens search from features view", func(t *testing.T) {
		app := &App{state: viewFeatures}
		app.features = featuresModel{
			inProgress: []FeatureItem{{Bean: &bean.Bean{ID: "beans-001", Title: "Test"}}},
		}
		app.width = 80
		app.height = 24

		_, _ = app.Update(tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("/")})

		assert.Equal(t, viewSearch, app.state)
	})

	t.Run("closeSearchMsg returns to previous view", func(t *testing.T) {
		app := &App{
			state:         viewSearch,
			previousState: viewFeatures,
		}

		_, _ = app.Update(closeSearchMsg{})

		assert.Equal(t, viewFeatures, app.state)
	})
}
```

**Step 2: Run all tests**

Run: `go test ./superbeans/... -v`
Expected: All tests pass

**Step 3: Commit**

```bash
git add superbeans/internal/tui/app_test.go
git commit -m "test(superbeans): add search flow integration tests

Refs: beans-iuaq"
```

---

### Task 3.2: Update beans and finalize

**Step 1: Mark design bean as completed**

```bash
beans update beans-qb4j -s completed
```

**Step 2: Mark plan bean as needs-review**

```bash
beans update beans-8axb -s completed --tag needs-review
```

**Step 3: Update feature bean status**

```bash
beans update beans-iuaq -s completed
```

---

## Summary

| Phase | Tasks | Description |
|-------|-------|-------------|
| 1 | 1.1-1.3 | Section limits with overflow indicators |
| 2 | 2.1-2.2 | Dedicated search view with live filtering |
| 3 | 3.1-3.2 | Integration tests and cleanup |

**Total tasks:** 7 implementation tasks + 1 cleanup task

**Key files modified:**
- `superbeans/internal/tui/features.go` - Section limits and overflow
- `superbeans/internal/tui/search.go` - New search view (created)
- `superbeans/internal/tui/app.go` - Integration

**Key files created:**
- `superbeans/internal/tui/search.go`
- `superbeans/internal/tui/search_test.go`
