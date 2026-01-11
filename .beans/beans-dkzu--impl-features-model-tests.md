---
# beans-dkzu
title: 'Impl: Features model tests'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:40:33Z
updated_at: 2026-01-04T23:55:06Z
parent: beans-ld6n
---

## Phase 4: Features Model Tests

**Files:**
- Create: `superbeans/internal/tui/features_test.go`

**Step 1: Create features_test.go with cursor navigation tests**

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

func TestFeaturesModel_CursorNavigation(t *testing.T) {
	tests := []struct {
		name           string
		initialCursor  int
		key            tea.KeyMsg
		expectedCursor int
	}{
		{
			name:           "j moves down",
			initialCursor:  0,
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("j")},
			expectedCursor: 1,
		},
		{
			name:           "down arrow moves down",
			initialCursor:  0,
			key:            tea.KeyMsg{Type: tea.KeyDown},
			expectedCursor: 1,
		},
		{
			name:           "k moves up",
			initialCursor:  1,
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("k")},
			expectedCursor: 0,
		},
		{
			name:           "up arrow moves up",
			initialCursor:  1,
			key:            tea.KeyMsg{Type: tea.KeyUp},
			expectedCursor: 0,
		},
		{
			name:           "k at top stays at 0",
			initialCursor:  0,
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("k")},
			expectedCursor: 0,
		},
		{
			name:           "j at bottom stays at last",
			initialCursor:  3, // 4 items, last index is 3
			key:            tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("j")},
			expectedCursor: 3,
		},
	}

	fixtures := newTestFixtures()
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			m := featuresModel{
				cursor: tt.initialCursor,
				items:  fixtures.all(),
			}

			updated, _ := m.Update(tt.key)

			assert.Equal(t, tt.expectedCursor, updated.cursor)
		})
	}
}

func TestFeaturesModel_SelectedFeature(t *testing.T) {
	fixtures := newTestFixtures()

	t.Run("returns feature at cursor", func(t *testing.T) {
		m := featuresModel{
			cursor: 0,
			items:  fixtures.all(),
		}

		selected := m.SelectedFeature()

		assert.NotNil(t, selected)
		assert.Equal(t, "beans-ip01", selected.Bean.ID)
	})

	t.Run("returns correct feature after cursor move", func(t *testing.T) {
		m := featuresModel{
			cursor: 1,
			items:  fixtures.all(),
		}

		selected := m.SelectedFeature()

		assert.NotNil(t, selected)
		assert.Equal(t, "beans-td01", selected.Bean.ID)
	})

	t.Run("returns nil when no items", func(t *testing.T) {
		m := featuresModel{
			cursor: 0,
			items:  []FeatureItem{},
		}

		selected := m.SelectedFeature()

		assert.Nil(t, selected)
	})

	t.Run("returns nil when cursor out of bounds", func(t *testing.T) {
		m := featuresModel{
			cursor: 10,
			items:  fixtures.all(),
		}

		selected := m.SelectedFeature()

		assert.Nil(t, selected)
	})
}

func TestFeaturesModel_FeaturesLoadedMsg(t *testing.T) {
	t.Run("updates model with loaded features", func(t *testing.T) {
		m := featuresModel{}
		fixtures := newTestFixtures()

		msg := featuresLoadedMsg{
			inProgress: fixtures.inProgress,
			todo:       fixtures.todo,
			done:       fixtures.done,
		}

		updated, _ := m.Update(msg)

		assert.Len(t, updated.inProgress, 1)
		assert.Len(t, updated.todo, 2)
		assert.Len(t, updated.done, 1)
		assert.Len(t, updated.items, 4)
	})

	t.Run("sets error on load failure", func(t *testing.T) {
		m := featuresModel{}
		msg := featuresLoadedMsg{
			err: assert.AnError,
		}

		updated, _ := m.Update(msg)

		assert.Equal(t, assert.AnError, updated.loadError)
	})

	t.Run("resets cursor if out of bounds", func(t *testing.T) {
		m := featuresModel{cursor: 10}
		fixtures := newTestFixtures()

		msg := featuresLoadedMsg{
			inProgress: fixtures.inProgress,
			todo:       fixtures.todo,
			done:       fixtures.done,
		}

		updated, _ := m.Update(msg)

		assert.Equal(t, 3, updated.cursor) // last valid index
	})
}
```

**Step 2: Run tests to verify they pass**

Run: `go test ./superbeans/internal/tui/ -run TestFeaturesModel -v`
Expected: All PASS

**Step 3: Commit**

```bash
git add superbeans/internal/tui/features_test.go
git commit -m "test(tui): add features model cursor navigation tests

- Test j/k and arrow key cursor movement
- Test boundary behavior (can't go past first/last)
- Test SelectedFeature() returns correct item
- Test featuresLoadedMsg handling

Refs: beans-ld6n"
```
