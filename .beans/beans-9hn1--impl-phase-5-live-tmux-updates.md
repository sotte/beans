---
# beans-9hn1
title: 'Impl: Phase 5 - Live tmux updates'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T21:45:32Z
updated_at: 2026-01-05T22:31:06Z
parent: beans-rmtd
---

## Phase 5: Live tmux updates

**Files:**
- Modify: `internal/supertui/app.go` (capture pane on refresh tick)

---

### Task 5.1: Add tmux content update message

**Step 1: Add message type to app.go**

```go
type tmuxContentUpdatedMsg struct {
	beanID  string
	content string
}
```

---

### Task 5.2: Capture pane content on session refresh

**Step 1: Update refreshSessionsMsg handler**

In `App.Update`, update the `refreshSessionsMsg` case:

```go
case refreshSessionsMsg:
	sessions := DetectSessionStates()
	a.features.updateSessions(sessions)
	
	// Capture tmux pane if tmux preview is active
	var captureCmd tea.Cmd
	if a.previewMode == PreviewTmux {
		var selectedBeanID string
		if a.state == viewFeatures {
			if f := a.features.SelectedFeature(); f != nil {
				selectedBeanID = f.Bean.ID
			}
		} else if a.state == viewDetail {
			selectedBeanID = a.detail.feature.Bean.ID
		}
		
		if selectedBeanID != "" {
			if sessionName := GetSessionName(selectedBeanID); sessionName != "" {
				captureCmd = func() tea.Msg {
					content, err := CapturePane(sessionName, 50)
					if err != nil {
						content = fmt.Sprintf("Error: %v", err)
					}
					return tmuxContentUpdatedMsg{beanID: selectedBeanID, content: content}
				}
			}
		}
	}
	
	return a, tea.Batch(
		tea.Every(1*time.Second, func(t time.Time) tea.Msg {
			return refreshSessionsMsg{}
		}),
		captureCmd,
	)
```

---

### Task 5.3: Handle tmux content update

**Step 1: Add handler for tmuxContentUpdatedMsg**

```go
case tmuxContentUpdatedMsg:
	// Update the cached tmux content in the appropriate model
	if a.state == viewFeatures {
		if f := a.features.SelectedFeature(); f != nil && f.Bean.ID == msg.beanID {
			a.features.tmuxContent = msg.content
		}
	} else if a.state == viewDetail {
		if a.detail.feature.Bean.ID == msg.beanID {
			a.detail.tmuxContent = msg.content
		}
	}
	return a, nil
```

---

### Task 5.4: Clear tmuxContent when switching views

**Step 1: Clear content when entering detail view**

In the `enter`/`right` key handler that creates detail view:
```go
case "enter", "right":
	if a.state == viewFeatures {
		if f := a.features.SelectedFeature(); f != nil {
			a.detail = newDetailModel(*f, a.width, a.height)
			a.detail.tmuxContent = "" // start fresh
			a.state = viewDetail
			return a, a.detail.Init()
		}
	}
```

**Step 2: Clear content when returning to features view**

In the `esc`/`left` key handler:
```go
case "esc", "left":
	if a.state == viewDetail {
		a.features.tmuxContent = "" // clear stale content
		a.state = viewFeatures
		return a, nil
	}
```

---

### Task 5.5: Test manually

**Step 1: Build and run**

Run: `mise build && ./beans super`

**Step 2: Test scenarios**

1. Select a feature with an active tmux session
2. Press `v` to get to tmux preview mode
3. Verify the tmux pane content appears
4. Verify it updates every second (watch for changes)
5. Press j/k to change selection - verify old content clears immediately
6. Select a feature without a session
7. Verify "No session — press a to attach" appears
8. Drill into detail view, verify tmux preview works there too
9. Return to features view, verify content reloads

**Step 3: Commit**

```
feat(supertui): add live tmux preview updates

- Capture tmux pane content every second when tmux preview is active
- Update cached content in features and detail models
- Clear stale content on selection change and view transitions

Refs: beans-rmtd
```
