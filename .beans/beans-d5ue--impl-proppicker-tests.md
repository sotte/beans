---
# beans-d5ue
title: 'Impl: PropPicker tests'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:40:39Z
updated_at: 2026-01-04T23:57:09Z
parent: beans-ld6n
---

## Phase 5: PropPicker Tests

**Files:**
- Create: `superbeans/internal/tui/proppicker_test.go`

**Step 1: Create proppicker_test.go with keyboard shortcut tests**

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

func TestPropPickerModel_StatusKeys(t *testing.T) {
	tests := []struct {
		key            string
		expectedStatus string
	}{
		{"i", "in-progress"},
		{"t", "todo"},
		{"d", "draft"},
		{"c", "completed"},
		{"x", "scrapped"},
	}

	for _, tt := range tests {
		t.Run(tt.key+" sets "+tt.expectedStatus, func(t *testing.T) {
			m := newPropPickerModel(&bean.Bean{ID: "test", Status: "todo"}, 80, 24)
			keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune(tt.key)}

			updated, _ := m.Update(keyMsg)

			assert.Equal(t, tt.expectedStatus, updated.status)
		})
	}
}

func TestPropPickerModel_PriorityKeys(t *testing.T) {
	tests := []struct {
		key              string
		expectedPriority string
	}{
		{"1", "critical"},
		{"2", "high"},
		{"3", "normal"},
		{"4", "low"},
		{"5", "deferred"},
	}

	for _, tt := range tests {
		t.Run(tt.key+" sets "+tt.expectedPriority, func(t *testing.T) {
			m := newPropPickerModel(&bean.Bean{ID: "test", Priority: "normal"}, 80, 24)
			keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune(tt.key)}

			updated, _ := m.Update(keyMsg)

			assert.Equal(t, tt.expectedPriority, updated.priority)
		})
	}
}

func TestPropPickerModel_ArtifactKeys(t *testing.T) {
	tests := []struct {
		key              string
		expectedArtifact string
	}{
		{"R", "research"},
		{"D", "design"},
		{"P", "plan"},
		{"I", "impl"},
	}

	for _, tt := range tests {
		t.Run(tt.key+" sets "+tt.expectedArtifact, func(t *testing.T) {
			m := newPropPickerModel(&bean.Bean{ID: "test"}, 80, 24)
			keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune(tt.key)}

			updated, _ := m.Update(keyMsg)

			assert.Equal(t, tt.expectedArtifact, updated.artifact)
		})
	}
}

func TestPropPickerModel_ArtifactToggle(t *testing.T) {
	t.Run("pressing same artifact key toggles off", func(t *testing.T) {
		m := newPropPickerModel(&bean.Bean{ID: "test"}, 80, 24)
		keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("R")}

		// Toggle on
		updated, _ := m.Update(keyMsg)
		assert.Equal(t, "research", updated.artifact)

		// Toggle off
		updated, _ = updated.Update(keyMsg)
		assert.Equal(t, "", updated.artifact)
	})
}

func TestPropPickerModel_ReviewToggle(t *testing.T) {
	t.Run("? toggles review flag", func(t *testing.T) {
		m := newPropPickerModel(&bean.Bean{ID: "test"}, 80, 24)
		keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("?")}

		// Toggle on
		updated, _ := m.Update(keyMsg)
		assert.True(t, updated.review)

		// Toggle off
		updated, _ = updated.Update(keyMsg)
		assert.False(t, updated.review)
	})
}

func TestPropPickerModel_EscCloses(t *testing.T) {
	m := newPropPickerModel(&bean.Bean{ID: "test"}, 80, 24)
	keyMsg := tea.KeyMsg{Type: tea.KeyEsc}

	_, cmd := m.Update(keyMsg)

	assert.NotNil(t, cmd)
	msg := cmd()
	_, ok := msg.(propPickerClosedMsg)
	assert.True(t, ok, "expected propPickerClosedMsg")
}

func TestPropPickerModel_EnterApplies(t *testing.T) {
	m := newPropPickerModel(&bean.Bean{ID: "test-bean", Status: "todo", Priority: "normal"}, 80, 24)
	m.status = "in-progress"
	m.priority = "high"
	m.artifact = "impl"
	m.review = true

	keyMsg := tea.KeyMsg{Type: tea.KeyEnter}
	_, cmd := m.Update(keyMsg)

	assert.NotNil(t, cmd)
	msg := cmd()
	applied, ok := msg.(propPickerAppliedMsg)
	assert.True(t, ok, "expected propPickerAppliedMsg")
	assert.Equal(t, "test-bean", applied.beanID)
	assert.Equal(t, "in-progress", applied.status)
	assert.Equal(t, "high", applied.priority)
	assert.Equal(t, "impl", applied.artifact)
	assert.True(t, applied.review)
}

func TestPropPickerModel_InitializesFromBean(t *testing.T) {
	b := &bean.Bean{
		ID:       "beans-test",
		Status:   "in-progress",
		Priority: "high",
		Tags:     []string{"artifact:design", "needs-review"},
	}

	m := newPropPickerModel(b, 80, 24)

	assert.Equal(t, "beans-test", m.beanID)
	assert.Equal(t, "in-progress", m.status)
	assert.Equal(t, "high", m.priority)
	assert.Equal(t, "design", m.artifact)
	assert.True(t, m.review)
}
```

**Step 2: Run tests to verify they pass**

Run: `go test ./superbeans/internal/tui/ -run TestPropPickerModel -v`
Expected: All PASS

**Step 3: Commit**

```bash
git add superbeans/internal/tui/proppicker_test.go
git commit -m "test(tui): add PropPicker keyboard shortcut tests

- Test status keys (i/t/d/c/x)
- Test priority keys (1-5)
- Test artifact keys (R/D/P/I) with toggle behavior
- Test review flag toggle (?)
- Test esc closes and enter applies
- Test initialization from bean

Refs: beans-ld6n"
```
