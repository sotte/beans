---
# beans-og1t
title: 'Impl: Phase 1 - PreviewMode enum and cycling'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T21:43:44Z
updated_at: 2026-01-05T22:04:47Z
parent: beans-rmtd
---

## Phase 1: PreviewMode enum and basic cycling

**Files:**
- Create: `internal/supertui/preview.go`
- Modify: `internal/supertui/app.go` (change showPreview to previewMode)
- Modify: `internal/supertui/detail.go` (read previewMode from App, don't store locally)
- Test: `internal/supertui/preview_test.go`

---

### Task 1.1: Write failing test for PreviewMode cycling

**Step 1: Create test file**

```go
// internal/supertui/preview_test.go
package supertui

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestPreviewMode_Cycle(t *testing.T) {
	tests := []struct {
		current  PreviewMode
		expected PreviewMode
	}{
		{PreviewOff, PreviewBody},
		{PreviewBody, PreviewTmux},
		{PreviewTmux, PreviewOff},
	}

	for _, tt := range tests {
		t.Run(tt.current.String()+"->"+tt.expected.String(), func(t *testing.T) {
			result := tt.current.Next()
			assert.Equal(t, tt.expected, result)
		})
	}
}

func TestPreviewMode_String(t *testing.T) {
	assert.Equal(t, "off", PreviewOff.String())
	assert.Equal(t, "body", PreviewBody.String())
	assert.Equal(t, "tmux", PreviewTmux.String())
}

func TestPreviewMode_ShowsPanel(t *testing.T) {
	assert.False(t, PreviewOff.ShowsPanel())
	assert.True(t, PreviewBody.ShowsPanel())
	assert.True(t, PreviewTmux.ShowsPanel())
}
```

**Step 2: Run test to verify it fails**

Run: `go test ./internal/supertui/ -run TestPreviewMode -v`
Expected: FAIL - PreviewMode not defined

---

### Task 1.2: Implement PreviewMode type

**Step 1: Create preview.go**

```go
// internal/supertui/preview.go
package supertui

// PreviewMode represents the current preview panel state
type PreviewMode int

const (
	PreviewOff  PreviewMode = iota // No preview panel
	PreviewBody                    // Show markdown body preview
	PreviewTmux                    // Show tmux pane content
)

// Next returns the next mode in the cycle: off → body → tmux → off
func (m PreviewMode) Next() PreviewMode {
	return (m + 1) % 3
}

// String returns the string representation of the mode
func (m PreviewMode) String() string {
	switch m {
	case PreviewBody:
		return "body"
	case PreviewTmux:
		return "tmux"
	default:
		return "off"
	}
}

// ShowsPanel returns true if this mode shows a preview panel
func (m PreviewMode) ShowsPanel() bool {
	return m != PreviewOff
}
```

**Step 2: Run test to verify it passes**

Run: `go test ./internal/supertui/ -run TestPreviewMode -v`
Expected: PASS

**Step 3: Commit**

```
feat(supertui): add PreviewMode enum for cycling preview states

- Add PreviewMode type with Off/Body/Tmux states
- Implement Next() for cycling through modes
- Add String() and ShowsPanel() helper methods

Refs: beans-rmtd
```

---

### Task 1.3: Update App to use PreviewMode

**Step 1: Update app.go**

In `App` struct, replace:
```go
showPreview   bool // global preview toggle state preserved across detail views
```
with:
```go
previewMode   PreviewMode // global preview mode preserved across views
```

**Step 2: Update previewToggledMsg**

Replace:
```go
type previewToggledMsg struct{ showPreview bool }
```
with:
```go
type previewToggledMsg struct{ mode PreviewMode }
```

**Step 3: Update handler in App.Update**

Replace:
```go
case previewToggledMsg:
	a.showPreview = msg.showPreview
	return a, nil
```
with:
```go
case previewToggledMsg:
	a.previewMode = msg.mode
	return a, nil
```

**Step 4: Update detail model creation - remove previewMode parameter**

The detail model will receive previewMode via messages, not constructor.
Update all `newDetailModel` calls to remove the last parameter:
```go
// Before:
a.detail = newDetailModel(*f, a.width, a.height, a.showPreview)
// After:
a.detail = newDetailModel(*f, a.width, a.height)
```

Do this in all 3 places (~lines 104, 215, 282).

---

### Task 1.4: Update detail model - remove local previewMode storage

**Step 1: Update detailModel struct - remove showPreview/previewMode**

The detail model should NOT store previewMode locally. Instead, it receives
the current mode via a new message type when rendering.

In `detail.go`, remove `showPreview bool` from the struct. Keep only:
```go
type detailModel struct {
	feature       FeatureItem
	cursor        int
	width         int
	height        int
	previewScroll int
	tmuxContent   string // cached tmux pane content (added in Phase 3)
}
```

**Step 2: Update newDetailModel signature**

```go
func newDetailModel(feature FeatureItem, width, height int) detailModel {
	return detailModel{
		feature: feature,
		cursor:  0,
		width:   width,
		height:  height,
	}
}
```

**Step 3: Add previewMode parameter to ViewWithStatus**

Change signature to receive mode from App:
```go
func (m detailModel) ViewWithStatus(statusMessage string, previewMode PreviewMode) string {
```

**Step 4: Update App.View to pass previewMode**

```go
case viewDetail:
	return a.detail.ViewWithStatus(a.statusMessage, a.previewMode)
```

**Step 5: Update v key handler in detail.Update**

The v key should send a message to App to cycle the mode:
```go
case "v":
	m.previewScroll = 0
	return m, func() tea.Msg {
		return cyclePreviewModeMsg{}
	}
```

**Step 6: Add cyclePreviewModeMsg and handler in App**

```go
type cyclePreviewModeMsg struct{}

// In App.Update:
case cyclePreviewModeMsg:
	a.previewMode = a.previewMode.Next()
	return a, nil
```

**Step 7: Update ViewWithStatus to use passed previewMode**

Replace `if m.showPreview {` with `if previewMode.ShowsPanel() {`
And use `previewMode` instead of `m.showPreview` throughout.

**Step 8: Run all tests**

Run: `go test ./internal/supertui/ -v`
Expected: PASS

**Step 9: Commit**

```
refactor(supertui): centralize PreviewMode in App

- App owns previewMode, child models receive it via method params
- v key sends cyclePreviewModeMsg to App
- Prevents state sync issues between App and child models

Refs: beans-rmtd
```
