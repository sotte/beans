---
# beans-ndib
title: 'Impl 2: Data Layer - Phase Inference & Session Detection'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T18:57:49Z
updated_at: 2026-01-01T19:45:04Z
parent: beans-kwuv
---

# Phase 2: Data Layer - Phase Inference & Session Detection

**Files:**
- Create: `superbeans/internal/tui/phase.go`
- Create: `superbeans/internal/tui/phase_test.go`
- Create: `superbeans/internal/tui/session.go`
- Create: `superbeans/internal/tui/session_test.go`
- Create: `superbeans/internal/tui/feature.go`

---

## Step 1: Write failing test for Phase type

Create `superbeans/internal/tui/phase_test.go`:

```go
package tui

import (
	"testing"

	"github.com/anthropics/beans/internal/bean"
	"github.com/stretchr/testify/assert"
)

func TestInferPhase_NoChildren(t *testing.T) {
	phase := InferPhase(nil)
	assert.Equal(t, "idea", phase.Name)
	assert.Equal(t, "○", phase.Symbol)
	assert.Empty(t, phase.Progress)
}

func TestInferPhase_ResearchInProgress(t *testing.T) {
	children := []*bean.Bean{
		{Status: "in-progress", Tags: []string{"artifact:research"}},
	}
	phase := InferPhase(children)
	assert.Equal(t, "research", phase.Name)
	assert.Equal(t, "◐", phase.Symbol)
}

func TestInferPhase_DesignInProgress(t *testing.T) {
	children := []*bean.Bean{
		{Status: "completed", Tags: []string{"artifact:research"}},
		{Status: "in-progress", Tags: []string{"artifact:design"}},
	}
	phase := InferPhase(children)
	assert.Equal(t, "design", phase.Name)
	assert.Equal(t, "◐", phase.Symbol)
}

func TestInferPhase_ImplPartial(t *testing.T) {
	children := []*bean.Bean{
		{Status: "completed", Tags: []string{"artifact:research"}},
		{Status: "completed", Tags: []string{"artifact:design"}},
		{Status: "completed", Tags: []string{"artifact:plan"}},
		{Status: "completed", Tags: []string{"artifact:impl"}},
		{Status: "in-progress", Tags: []string{"artifact:impl"}},
		{Status: "todo", Tags: []string{"artifact:impl"}},
	}
	phase := InferPhase(children)
	assert.Equal(t, "impl", phase.Name)
	assert.Equal(t, "1/3", phase.Progress)
	assert.Equal(t, "◐", phase.Symbol)
}

func TestInferPhase_AllComplete(t *testing.T) {
	children := []*bean.Bean{
		{Status: "completed", Tags: []string{"artifact:impl"}},
		{Status: "completed", Tags: []string{"artifact:impl"}},
	}
	phase := InferPhase(children)
	assert.Equal(t, "complete", phase.Name)
	assert.Equal(t, "✓", phase.Symbol)
}
```

---

## Step 2: Run test to verify it fails

```bash
go test ./superbeans/internal/tui/... -v -run TestInferPhase
```

Expected: FAIL - undefined: InferPhase

---

## Step 3: Implement Phase type and InferPhase

Create `superbeans/internal/tui/phase.go`:

**Note:** This uses `HasArtifactTag` from feature.go (Step 9). Create feature.go first, or create both files before running tests.

```go
package tui

import (
	"fmt"

	"github.com/anthropics/beans/internal/bean"
)

// Phase represents where a feature is in the SDLC pipeline
type Phase struct {
	Name     string // "idea", "research", "design", "plan", "impl", "complete"
	Progress string // "3/5" for impl phase, empty otherwise
	Symbol   string // "○", "◐", "✓"
}

// InferPhase determines the current phase of a feature based on its children's artifact tags
func InferPhase(children []*bean.Bean) Phase {
	if len(children) == 0 {
		return Phase{Name: "idea", Symbol: "○"}
	}

	// Check for in-progress phases in order
	phases := []string{"research", "design", "plan"}
	for _, phaseName := range phases {
		for _, child := range children {
			if HasArtifactTag(child, phaseName) && child.Status == "in-progress" {
				return Phase{Name: phaseName, Symbol: "◐"}
			}
		}
	}

	// Check for impl phases
	var implBeans []*bean.Bean
	for _, child := range children {
		if HasArtifactTag(child, "impl") {
			implBeans = append(implBeans, child)
		}
	}

	if len(implBeans) > 0 {
		completed := 0
		for _, impl := range implBeans {
			if impl.Status == "completed" {
				completed++
			}
		}
		if completed == len(implBeans) {
			return Phase{Name: "complete", Symbol: "✓"}
		}
		return Phase{
			Name:     "impl",
			Progress: fmt.Sprintf("%d/%d", completed, len(implBeans)),
			Symbol:   "◐",
		}
	}

	// Check if all children are completed
	allComplete := true
	for _, child := range children {
		if child.Status != "completed" {
			allComplete = false
			break
		}
	}
	if allComplete {
		return Phase{Name: "complete", Symbol: "✓"}
	}

	return Phase{Name: "idea", Symbol: "○"}
}

// String renders the phase for display
func (p Phase) String() string {
	if p.Progress != "" {
		return fmt.Sprintf("%s %s %s", p.Symbol, p.Name, p.Progress)
	}
	return fmt.Sprintf("%s %s", p.Symbol, p.Name)
}
```

---

## Step 4: Run tests to verify they pass

```bash
go test ./superbeans/internal/tui/... -v -run TestInferPhase
```

Expected: PASS

---

## Step 5: Write failing test for session detection

Add to `superbeans/internal/tui/session_test.go`:

```go
package tui

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestParseSessionOutput(t *testing.T) {
	output := `beans-kxob-impl
beans-kwuv-feature
other-session
`
	sessions := ParseSessionOutput(output)

	assert.True(t, sessions["beans-kxob"])
	assert.True(t, sessions["beans-kwuv"])
	assert.False(t, sessions["other"])
	assert.False(t, sessions["beans-xxxx"])
}

func TestParseSessionOutput_Empty(t *testing.T) {
	sessions := ParseSessionOutput("")
	assert.Empty(t, sessions)
}
```

---

## Step 6: Run test to verify it fails

```bash
go test ./superbeans/internal/tui/... -v -run TestParseSession
```

Expected: FAIL - undefined: ParseSessionOutput

---

## Step 7: Implement session detection

Create `superbeans/internal/tui/session.go`:

```go
package tui

import (
	"os/exec"
	"regexp"
	"strings"
)

// beanIDPattern matches bean IDs like "beans-kxob" at the start of session names
var beanIDPattern = regexp.MustCompile(`^(beans-[a-z0-9]+)`)

// ParseSessionOutput extracts bean IDs from tmux session names
func ParseSessionOutput(output string) map[string]bool {
	sessions := make(map[string]bool)
	lines := strings.Split(strings.TrimSpace(output), "\n")

	for _, line := range lines {
		line = strings.TrimSpace(line)
		if line == "" {
			continue
		}
		if match := beanIDPattern.FindString(line); match != "" {
			sessions[match] = true
		}
	}

	return sessions
}

// DetectSessions runs tmux list-sessions and returns a map of bean IDs with active sessions
func DetectSessions() map[string]bool {
	cmd := exec.Command("tmux", "list-sessions", "-F", "#{session_name}")
	output, err := cmd.Output()
	if err != nil {
		// tmux not running or no sessions - return empty map
		return make(map[string]bool)
	}
	return ParseSessionOutput(string(output))
}

// GetSessionName returns the session name for a bean ID, or empty string if no session
func GetSessionName(beanID string) string {
	cmd := exec.Command("tmux", "list-sessions", "-F", "#{session_name}")
	output, err := cmd.Output()
	if err != nil {
		return ""
	}

	lines := strings.Split(strings.TrimSpace(string(output)), "\n")
	for _, line := range lines {
		if strings.HasPrefix(line, beanID) {
			return line
		}
	}
	return ""
}
```

---

## Step 8: Run tests to verify they pass

```bash
go test ./superbeans/internal/tui/... -v -run TestParseSession
```

Expected: PASS

---

## Step 9: Create FeatureItem type and shared helpers

Create `superbeans/internal/tui/feature.go`:

```go
package tui

import (
	"strings"

	"github.com/anthropics/beans/internal/bean"
)

// FeatureItem wraps a bean with computed SDLC state
type FeatureItem struct {
	Bean        *bean.Bean
	Phase       Phase
	HasSession  bool
	SessionName string
	NeedsReview bool // Any child has needs-review tag
	Children    []*bean.Bean
}

// HasTag checks if a bean has a specific tag
// This is used by phase inference, detail view, and review detection
func HasTag(b *bean.Bean, tag string) bool {
	for _, t := range b.Tags {
		if t == tag {
			return true
		}
	}
	return false
}

// HasArtifactTag checks if a bean has a specific artifact tag (e.g., "impl" checks for "artifact:impl")
func HasArtifactTag(b *bean.Bean, artifact string) bool {
	return HasTag(b, "artifact:"+artifact)
}

// GetArtifactType returns the artifact type from tags, or empty string if none
func GetArtifactType(b *bean.Bean) string {
	for _, tag := range b.Tags {
		if strings.HasPrefix(tag, "artifact:") {
			return strings.TrimPrefix(tag, "artifact:")
		}
	}
	return ""
}

// HasNeedsReview checks if any of the given beans have the needs-review tag
func HasNeedsReview(beans []*bean.Bean) bool {
	for _, b := range beans {
		if HasTag(b, "needs-review") {
			return true
		}
	}
	return false
}

// NewFeatureItem creates a FeatureItem with computed phase and review status
func NewFeatureItem(feature *bean.Bean, children []*bean.Bean, sessions map[string]bool) FeatureItem {
	return FeatureItem{
		Bean:        feature,
		Phase:       InferPhase(children),
		HasSession:  sessions[feature.ID],
		SessionName: GetSessionName(feature.ID),
		NeedsReview: HasNeedsReview(children),
		Children:    children,
	}
}
```

**Note:** `HasTag`, `HasArtifactTag`, and `GetArtifactType` are consolidated here for use by phase.go and detail.go.

---

## Step 10: Commit

```bash
git add superbeans/internal/tui/phase.go superbeans/internal/tui/phase_test.go \
        superbeans/internal/tui/session.go superbeans/internal/tui/session_test.go \
        superbeans/internal/tui/feature.go
git commit -m "feat(superbeans): add phase inference and session detection

- Add Phase type with InferPhase() for SDLC pipeline state
- Add tmux session detection with ParseSessionOutput()
- Add FeatureItem type combining bean with computed state
- Include tests for phase inference and session parsing

Refs: beans-ndib"
```

---

## Checklist

- [ ] phase_test.go created with tests
- [ ] phase.go implemented and tests pass
- [ ] session_test.go created with tests
- [ ] session.go implemented and tests pass
- [ ] feature.go with FeatureItem type
- [ ] All tests pass
- [ ] Committed