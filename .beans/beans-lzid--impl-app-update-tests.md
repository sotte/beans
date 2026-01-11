---
# beans-lzid
title: 'Impl: App Update() tests'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:40:28Z
updated_at: 2026-01-04T23:53:25Z
parent: beans-ld6n
---

## Phase 3: App Update() Tests

**Files:**
- Modify: `superbeans/internal/tui/app_test.go`

**Step 1: Add quit key tests**

Append to `app_test.go`:

```go
func TestApp_Update_QuitKeys(t *testing.T) {
	tests := []struct {
		name string
		key  tea.KeyMsg
	}{
		{
			name: "q quits",
			key:  tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("q")},
		},
		{
			name: "ctrl+c quits",
			key:  tea.KeyMsg{Type: tea.KeyCtrlC},
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			app := createTestApp(viewFeatures)
			_, cmd := app.Update(tt.key)

			// Should return tea.Quit command
			assert.NotNil(t, cmd)
		})
	}
}
```

**Step 2: Run test to verify it passes**

Run: `go test ./superbeans/internal/tui/ -run TestApp_Update_QuitKeys -v`
Expected: PASS

**Step 3: Add navigation tests**

Append to `app_test.go`:

```go
func TestApp_Update_Navigation(t *testing.T) {
	t.Run("enter navigates from features to detail", func(t *testing.T) {
		app := createTestApp(viewFeatures)
		keyMsg := tea.KeyMsg{Type: tea.KeyEnter}

		model, cmd := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewDetail, updatedApp.state)
		assert.NotNil(t, cmd) // detail.Init() is called
	})

	t.Run("right navigates from features to detail", func(t *testing.T) {
		app := createTestApp(viewFeatures)
		keyMsg := tea.KeyMsg{Type: tea.KeyRight}

		model, _ := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewDetail, updatedApp.state)
	})

	t.Run("esc returns from detail to features", func(t *testing.T) {
		app := createTestApp(viewDetail)
		keyMsg := tea.KeyMsg{Type: tea.KeyEsc}

		model, _ := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewFeatures, updatedApp.state)
	})

	t.Run("left returns from detail to features", func(t *testing.T) {
		app := createTestApp(viewDetail)
		keyMsg := tea.KeyMsg{Type: tea.KeyLeft}

		model, _ := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewFeatures, updatedApp.state)
	})

	t.Run("esc in features view does nothing", func(t *testing.T) {
		app := createTestApp(viewFeatures)
		keyMsg := tea.KeyMsg{Type: tea.KeyEsc}

		model, _ := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewFeatures, updatedApp.state)
	})
}
```

**Step 4: Run navigation tests**

Run: `go test ./superbeans/internal/tui/ -run TestApp_Update_Navigation -v`
Expected: PASS

**Step 5: Add modal lifecycle tests**

Append to `app_test.go`:

```go
func TestApp_Update_ModalLifecycle(t *testing.T) {
	t.Run("p opens prop picker and sets previousState", func(t *testing.T) {
		app := createTestApp(viewFeatures)
		keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("p")}

		model, cmd := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewPropPicker, updatedApp.state)
		assert.Equal(t, viewFeatures, updatedApp.previousState)
		assert.NotNil(t, cmd) // propPicker.Init() is called
	})

	t.Run("p from detail sets detail as previousState", func(t *testing.T) {
		app := createTestApp(viewDetail)
		keyMsg := tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("p")}

		model, _ := app.Update(keyMsg)
		updatedApp := model.(*App)

		assert.Equal(t, viewPropPicker, updatedApp.state)
		assert.Equal(t, viewDetail, updatedApp.previousState)
	})

	t.Run("propPickerClosedMsg restores previousState", func(t *testing.T) {
		app := createTestApp(viewFeatures)
		app.state = viewPropPicker
		app.previousState = viewDetail

		model, _ := app.Update(propPickerClosedMsg{})
		updatedApp := model.(*App)

		assert.Equal(t, viewDetail, updatedApp.state)
	})
}
```

**Step 6: Run modal tests**

Run: `go test ./superbeans/internal/tui/ -run TestApp_Update_ModalLifecycle -v`
Expected: PASS

**Step 7: Add WindowSizeMsg broadcast test**

Append to `app_test.go`:

```go
func TestApp_Update_WindowSizeMsg(t *testing.T) {
	app := createTestApp(viewFeatures)
	sizeMsg := tea.WindowSizeMsg{Width: 120, Height: 40}

	model, _ := app.Update(sizeMsg)
	updatedApp := model.(*App)

	assert.Equal(t, 120, updatedApp.width)
	assert.Equal(t, 40, updatedApp.height)
	assert.Equal(t, 120, updatedApp.features.width)
	assert.Equal(t, 40, updatedApp.features.height)
	assert.Equal(t, 120, updatedApp.detail.width)
	assert.Equal(t, 40, updatedApp.detail.height)
}
```

**Step 8: Run all app tests**

Run: `go test ./superbeans/internal/tui/ -run TestApp -v`
Expected: All PASS

**Step 9: Commit**

```bash
git add superbeans/internal/tui/app_test.go
git commit -m "test(tui): add App Update() tests for navigation and modal lifecycle

- Test quit keys (q, ctrl+c)
- Test view navigation (enter, esc, left, right)
- Test modal open/close with previousState preservation
- Test WindowSizeMsg broadcast to child models

Refs: beans-ld6n"
```
