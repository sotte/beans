---
# beans-p7j0
title: 'Impl: Phase 2 - CapturePane function'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-05T21:44:06Z
updated_at: 2026-01-05T22:08:09Z
parent: beans-rmtd
---

## Phase 2: CapturePane function

**Files:**
- Modify: `internal/supertui/session.go` (add CapturePane function)
- Modify: `internal/supertui/session_test.go` (add tests)

---

### Task 2.1: Implement CapturePane

**Step 1: Add function to session.go**

```go
// CapturePane captures the last N lines from a tmux session's pane.
// Uses tmux capture-pane with 1-based window numbering.
// Returns the captured content or an error if the session doesn't exist.
func CapturePane(sessionName string, lines int) (string, error) {
	// Use -p to print to stdout, -S to specify start line (negative = history)
	// -t session:1 targets window 1 (tmux uses 1-based by default)
	cmd := exec.Command("tmux", "capture-pane", "-t", sessionName+":1", "-p", "-S", fmt.Sprintf("-%d", lines))
	output, err := cmd.Output()
	if err != nil {
		return "", fmt.Errorf("failed to capture pane: %w", err)
	}
	return string(output), nil
}
```

**Step 2: Add import for fmt if needed**

```go
import (
	"fmt"
	"os/exec"
	// ... existing imports
)
```

**Step 3: Run existing tests to verify no regressions**

Run: `go test ./internal/supertui/ -v`
Expected: PASS

**Step 4: Commit**

```
feat(supertui): add CapturePane function for tmux pane capture

- Captures last N lines from a tmux session's pane
- Uses tmux capture-pane command with 1-based window targeting
- Returns error if session doesn't exist

Refs: beans-rmtd
```

Note: No unit test for CapturePane since it requires a real tmux session. Will be tested manually in Phase 5.
