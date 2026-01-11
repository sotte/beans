---
# beans-et6o
title: 'Impl: SessionAsk style and rendering'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T00:51:57Z
updated_at: 2026-01-05T02:57:54Z
parent: beans-1xgc
---

# Impl: SessionAsk style and rendering

**Files:**
- Modify: `superbeans/internal/tui/styles.go:54-62` (add StyleSessionAsk)
- Modify: `superbeans/internal/tui/features.go:373-385` (update rendering)
- Modify: `superbeans/internal/tui/detail.go:189-201` (update rendering)
- Test: `superbeans/internal/tui/feature_test.go`

---

### Step 1: Write failing test for SessionAsk in NewFeatureItem

Add test case to `feature_test.go`:

```go
t.Run("feature with ask session", func(t *testing.T) {
	feature := &bean.Bean{ID: "beans-xyz", Title: "Test Feature"}
	sessions := map[string]SessionState{"beans-xyz": SessionAsk}
	children := []*bean.Bean{}

	item := NewFeatureItem(feature, children, sessions)

	assert.Equal(t, SessionAsk, item.SessionState)
})
```

### Step 2: Run test to verify it fails

Run: `go test ./superbeans/internal/tui/ -run "TestNewFeatureItem/feature_with_ask" -v`
Expected: FAIL with "undefined: SessionAsk"

### Step 3: Run test again after Step 1 of phase 1 is complete

Run: `go test ./superbeans/internal/tui/ -run "TestNewFeatureItem/feature_with_ask" -v`
Expected: PASS (SessionAsk already added in phase 1)

### Step 4: Add StyleSessionAsk to styles.go

In `styles.go`, add after `StyleSessionIdle`:

```go
	StyleSessionIdle = lipgloss.NewStyle().
			Foreground(ColorAmber)

	StyleSessionAsk = lipgloss.NewStyle().
			Foreground(ColorAmber)

	StyleSessionNone = lipgloss.NewStyle().
```

### Step 5: Update rendering in features.go

In `features.go`, update the session indicator switch in `renderFeatureRow`:

```go
	// Session indicator with state
	var sessionStr string
	var sessionStyle lipgloss.Style
	switch item.SessionState {
	case SessionWorking:
		sessionStr = "● work"
		sessionStyle = StyleSessionWorking
	case SessionAsk:
		sessionStr = "? ask"
		sessionStyle = StyleSessionAsk
	case SessionIdle:
		sessionStr = "⏸ idle"
		sessionStyle = StyleSessionIdle
	default:
		sessionStr = "○ none"
		sessionStyle = StyleSessionNone
	}
```

### Step 6: Update rendering in detail.go

In `detail.go`, update the session indicator switch in `ViewWithStatus`:

```go
	// Header: SUPERBEANS → Feature Title  [session state]
	var sessionIndicator string
	var sessionStyle lipgloss.Style
	switch m.feature.SessionState {
	case SessionWorking:
		sessionIndicator = "● work"
		sessionStyle = StyleSessionWorking
	case SessionAsk:
		sessionIndicator = "? ask"
		sessionStyle = StyleSessionAsk
	case SessionIdle:
		sessionIndicator = "⏸ idle"
		sessionStyle = StyleSessionIdle
	default:
		sessionIndicator = "○ none"
		sessionStyle = StyleSessionNone
	}
```

### Step 7: Run all TUI tests

Run: `go test ./superbeans/internal/tui/ -v`
Expected: All PASS

### Step 8: Commit

```bash
git add superbeans/internal/tui/styles.go superbeans/internal/tui/features.go superbeans/internal/tui/detail.go superbeans/internal/tui/feature_test.go
git commit -m "feat(superbeans): add SessionAsk style and rendering

- Add StyleSessionAsk style (amber color)
- Render '? ask' indicator for SessionAsk state in features list
- Render '? ask' indicator for SessionAsk state in detail view
- Add test for SessionAsk session state

Refs: beans-1xgc"
```
