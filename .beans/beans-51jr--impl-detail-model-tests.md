---
# beans-51jr
title: 'Impl: Detail model tests'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:40:43Z
updated_at: 2026-01-04T23:59:14Z
parent: beans-ld6n
---

## Phase 6: Detail Model Tests

**Files:**
- Create: `superbeans/internal/tui/detail_test.go`

**Step 1: Create detail_test.go with cursor navigation tests**

```go
package tui

import (
	"testing"

	tea "github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"
	"github.com/hmans/beans/internal/bean"
	"github.com/muesli/termenv"
	"github.com/stretchr/testify/assert"
)

func init() {
	lipgloss.SetColorProfile(termenv.Ascii)
}

func createTestDetailModel() detailModel {
	parent := &bean.Bean{ID: "beans-parent", Title: "Parent Feature", Status: "in-progress"}
	child1 := &bean.Bean{ID: "beans-child1", Title: "Research", Tags: []string{"artifact:research"}, Status: "completed"}
	child2 := &bean.Bean{ID: "beans-child2", Title: "Implementation", Tags: []string{"artifact:impl"}, Status: "todo"}

	feature := FeatureItem{
		Bean:     parent,
		Children: []*bean.Bean{child1, child2},
	}

	return newDetailModel(feature, 80, 24)
}

func TestDetailModel_CursorNavigation(t *testing.T) {
	tests := []struct {
		name           string
		initialCursor  int
		key            tea.KeyMsg
		expectedCursor int
	}{
		{
			name:           "j moves down from parent",
			initialCursor:  0,
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("j")},
			expectedCursor: 1,
		},
		{
			name:           "down moves down from parent",
			initialCursor:  0,
			key:            tea.KeyMsg{Type: tea.KeyDown},
			expectedCursor: 1,
		},
		{
			name:           "k moves up to parent",
			initialCursor:  1,
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("k")},
			expectedCursor: 0,
		},
		{
			name:           "up moves up to parent",
			initialCursor:  1,
			key:            tea.KeyMsg{Type: tea.KeyUp},
			expectedCursor: 0,
		},
		{
			name:           "k at parent stays at 0",
			initialCursor:  0,
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("k")},
			expectedCursor: 0,
		},
		{
			name:           "j at last child stays at last",
			initialCursor:  2, // parent + 2 children, last index is 2
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("j")},
			expectedCursor: 2,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			m := createTestDetailModel()
			m.cursor = tt.initialCursor

			updated, _ := m.Update(tt.key)

			assert.Equal(t, tt.expectedCursor, updated.cursor)
		})
	}
}

func TestDetailModel_SelectedBean(t *testing.T) {
	t.Run("cursor 0 returns parent", func(t *testing.T) {
		m := createTestDetailModel()
		m.cursor = 0

		selected := m.SelectedBean()

		assert.NotNil(t, selected)
		assert.Equal(t, "beans-parent", selected.ID)
	})

	t.Run("cursor 1 returns first child (sorted by artifact)", func(t *testing.T) {
		m := createTestDetailModel()
		m.cursor = 1

		selected := m.SelectedBean()

		assert.NotNil(t, selected)
		// Children sorted: research (0) before impl (3)
		assert.Equal(t, "beans-child1", selected.ID)
	})

	t.Run("cursor 2 returns second child", func(t *testing.T) {
		m := createTestDetailModel()
		m.cursor = 2

		selected := m.SelectedBean()

		assert.NotNil(t, selected)
		assert.Equal(t, "beans-child2", selected.ID)
	})
}

func TestDetailModel_SelectedChild(t *testing.T) {
	t.Run("returns nil when parent selected", func(t *testing.T) {
		m := createTestDetailModel()
		m.cursor = 0

		selected := m.SelectedChild()

		assert.Nil(t, selected)
	})

	t.Run("returns child when child selected", func(t *testing.T) {
		m := createTestDetailModel()
		m.cursor = 1

		selected := m.SelectedChild()

		assert.NotNil(t, selected)
		assert.Equal(t, "beans-child1", selected.ID)
	})
}

func TestDetailModel_NoChildren(t *testing.T) {
	parent := &bean.Bean{ID: "beans-parent", Title: "Parent Feature"}
	feature := FeatureItem{
		Bean:     parent,
		Children: []*bean.Bean{},
	}
	m := newDetailModel(feature, 80, 24)

	t.Run("cursor stays at 0 with no children", func(t *testing.T) {
		keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("j")}

		updated, _ := m.Update(keyMsg)

		assert.Equal(t, 0, updated.cursor)
	})

	t.Run("SelectedBean returns parent", func(t *testing.T) {
		selected := m.SelectedBean()

		assert.NotNil(t, selected)
		assert.Equal(t, "beans-parent", selected.ID)
	})
}
```

**Step 2: Run tests to verify they pass**

Run: `go test ./superbeans/internal/tui/ -run TestDetailModel -v`
Expected: All PASS

**Step 3: Run all TUI tests**

Run: `go test ./superbeans/internal/tui/ -v`
Expected: All PASS

**Step 4: Commit**

```bash
git add superbeans/internal/tui/detail_test.go
git commit -m "test(tui): add detail model cursor navigation tests

- Test j/k and arrow key cursor movement
- Test boundary behavior (parent and last child)
- Test SelectedBean() for parent vs child selection
- Test SelectedChild() returns nil for parent
- Test edge case with no children

Refs: beans-ld6n"
```

**Step 5: Final verification**

Run: `go test ./superbeans/internal/tui/ -v`
Expected: All tests pass (existing + new)
