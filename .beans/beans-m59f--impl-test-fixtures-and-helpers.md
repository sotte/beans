---
# beans-m59f
title: 'Impl: Test fixtures and helpers'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:40:22Z
updated_at: 2026-01-04T23:49:40Z
parent: beans-ld6n
---

## Phase 2: Test Fixtures and Helpers

**Files:**
- Create: `superbeans/internal/tui/app_test.go`

**Step 1: Create app_test.go with test fixtures**

```go
package tui

import (
	"testing"

	"github.com/charmbracelet/lipgloss"
	"github.com/hmans/beans/internal/bean"
	"github.com/muesli/termenv"
	"github.com/stretchr/testify/assert"
)

func init() {
	// Set consistent color profile for CI compatibility
	lipgloss.SetColorProfile(termenv.Ascii)
}

// testFixtures provides reusable test data for TUI tests
type testFixtures struct {
	inProgress []FeatureItem
	todo       []FeatureItem
	done       []FeatureItem
}

func newTestFixtures() testFixtures {
	return testFixtures{
		inProgress: []FeatureItem{
			{
				Bean:  &bean.Bean{ID: "beans-ip01", Title: "In Progress Feature", Status: "in-progress"},
				Phase: Phase{Name: "impl", Symbol: "◐"},
			},
		},
		todo: []FeatureItem{
			{
				Bean:  &bean.Bean{ID: "beans-td01", Title: "Todo Feature 1", Status: "todo"},
				Phase: Phase{Name: "idea", Symbol: "○"},
			},
			{
				Bean:  &bean.Bean{ID: "beans-td02", Title: "Todo Feature 2", Status: "todo"},
				Phase: Phase{Name: "research", Symbol: "◐"},
			},
		},
		done: []FeatureItem{
			{
				Bean:  &bean.Bean{ID: "beans-dn01", Title: "Done Feature", Status: "completed"},
				Phase: Phase{Name: "implemented", Symbol: "✓"},
			},
		},
	}
}

func (f testFixtures) all() []FeatureItem {
	var all []FeatureItem
	all = append(all, f.inProgress...)
	all = append(all, f.todo...)
	all = append(all, f.done...)
	return all
}

// createTestApp creates an App for testing with the given view state
func createTestApp(state viewState) *App {
	fixtures := newTestFixtures()
	app := &App{
		state:  state,
		width:  80,
		height: 24,
	}
	app.features = featuresModel{
		cursor:     0,
		width:      80,
		height:     24,
		inProgress: fixtures.inProgress,
		todo:       fixtures.todo,
		done:       fixtures.done,
		items:      fixtures.all(),
	}
	if state == viewDetail {
		app.detail = newDetailModel(fixtures.inProgress[0], 80, 24)
	}
	return app
}

// TestFixturesExist verifies the test fixtures are correctly set up
func TestFixturesExist(t *testing.T) {
	fixtures := newTestFixtures()

	assert.Len(t, fixtures.inProgress, 1)
	assert.Len(t, fixtures.todo, 2)
	assert.Len(t, fixtures.done, 1)
	assert.Len(t, fixtures.all(), 4)
}

// TestCreateTestApp verifies the test app factory
func TestCreateTestApp(t *testing.T) {
	t.Run("creates app in features view", func(t *testing.T) {
		app := createTestApp(viewFeatures)
		assert.Equal(t, viewFeatures, app.state)
		assert.Equal(t, 80, app.width)
		assert.Len(t, app.features.items, 4)
	})

	t.Run("creates app in detail view", func(t *testing.T) {
		app := createTestApp(viewDetail)
		assert.Equal(t, viewDetail, app.state)
		assert.NotNil(t, app.detail.feature.Bean)
	})
}
```

**Step 2: Run test to verify fixtures work**

Run: `go test ./superbeans/internal/tui/ -run TestFixtures -v`
Expected: PASS

**Step 3: Commit**

```bash
git add superbeans/internal/tui/app_test.go
git commit -m "test(tui): add test fixtures and helpers for TUI tests

- Add testFixtures struct with sample FeatureItems
- Add createTestApp() helper for different view states
- Set CI-compatible color profile in init()

Refs: beans-ld6n"
```
