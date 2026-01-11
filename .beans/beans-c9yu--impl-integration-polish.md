---
# beans-c9yu
title: 'Impl 5: Integration & Polish'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T18:57:51Z
updated_at: 2026-01-01T19:54:39Z
parent: beans-kwuv
---

# Phase 5: Integration & Polish

**Files:**
- Modify: `superbeans/internal/tui/app.go` (complete rewrite with all features)
- Modify: `superbeans/internal/tui/session.go` (add attach command)
- Modify: `superbeans/cmd/root.go` (add file watching)
- Modify: `.mise.toml` (add build tasks)

**Note:** This phase shows complete file contents to avoid ambiguity.

---

## Step 1: Complete session.go with attach command

Replace `superbeans/internal/tui/session.go` with:

```go
package tui

import (
	"os/exec"
	"regexp"
	"strings"

	tea "github.com/charmbracelet/bubbletea"
)

// Message types for session operations
type sessionAttachedMsg struct {
	beanID string
}

type sessionAttachFailedMsg struct {
	beanID string
	err    error
}

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

// AttachToSession attaches to a tmux session for a bean
func AttachToSession(beanID string) tea.Cmd {
	return tea.ExecProcess(exec.Command("tmux", "attach", "-t", beanID), func(err error) tea.Msg {
		if err != nil {
			return sessionAttachFailedMsg{beanID: beanID, err: err}
		}
		return sessionAttachedMsg{beanID: beanID}
	})
}
```

---

## Step 2: Complete app.go with all integration

Replace `superbeans/internal/tui/app.go` with:

```go
package tui

import (
	"fmt"
	"time"

	"github.com/anthropics/beans/internal/beancore"
	"github.com/anthropics/beans/internal/config"
	"github.com/anthropics/beans/internal/graph"
	tea "github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"
)

type viewState int

const (
	viewFeatures viewState = iota
	viewDetail
)

// Message types
type beansChangedMsg struct{}
type refreshSessionsMsg struct{}
type clearStatusMsg struct{}

type App struct {
	state         viewState
	features      featuresModel
	detail        detailModel
	resolver      *graph.Resolver
	config        *config.Config
	core          *beancore.Core
	program       *tea.Program
	width         int
	height        int
	statusMessage string
}

func New(core *beancore.Core, cfg *config.Config) *App {
	resolver := &graph.Resolver{Core: core}
	return &App{
		state:    viewFeatures,
		resolver: resolver,
		config:   cfg,
		core:     core,
	}
}

func (a *App) Init() tea.Cmd {
	a.features = newFeaturesModel(a.resolver, a.width, a.height)
	return tea.Batch(
		a.features.Init(),
		tea.Every(5*time.Second, func(t time.Time) tea.Msg {
			return refreshSessionsMsg{}
		}),
	)
}

func (a *App) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		case "q", "ctrl+c":
			return a, tea.Quit
		case "esc":
			if a.state == viewDetail {
				a.state = viewFeatures
				return a, nil
			}
		case "enter":
			if a.state == viewFeatures {
				if f := a.features.SelectedFeature(); f != nil {
					a.detail = newDetailModel(*f, a.width, a.height)
					a.state = viewDetail
					return a, a.detail.Init()
				}
			}
		case "a":
			var beanID string
			if a.state == viewFeatures {
				if f := a.features.SelectedFeature(); f != nil {
					beanID = f.Bean.ID
				}
			} else if a.state == viewDetail {
				beanID = a.detail.feature.Bean.ID
			}
			if beanID != "" {
				return a, AttachToSession(beanID)
			}
		case "r":
			return a, a.features.loadFeatures
		}

	case tea.WindowSizeMsg:
		a.width = msg.Width
		a.height = msg.Height
		a.features.width = msg.Width
		a.features.height = msg.Height
		a.detail.width = msg.Width
		a.detail.height = msg.Height

	case beansChangedMsg:
		return a, a.features.loadFeatures

	case refreshSessionsMsg:
		sessions := DetectSessions()
		a.features.updateSessions(sessions)
		return a, tea.Every(5*time.Second, func(t time.Time) tea.Msg {
			return refreshSessionsMsg{}
		})

	case sessionAttachFailedMsg:
		a.statusMessage = fmt.Sprintf("No session for %s", msg.beanID)
		return a, tea.Tick(3*time.Second, func(t time.Time) tea.Msg {
			return clearStatusMsg{}
		})

	case clearStatusMsg:
		a.statusMessage = ""
		return a, nil
	}

	// Route to active view
	switch a.state {
	case viewFeatures:
		var cmd tea.Cmd
		a.features, cmd = a.features.Update(msg)
		return a, cmd
	case viewDetail:
		var cmd tea.Cmd
		a.detail, cmd = a.detail.Update(msg)
		return a, cmd
	}

	return a, nil
}

func (a *App) View() string {
	var view string
	switch a.state {
	case viewFeatures:
		view = a.features.View()
	case viewDetail:
		view = a.detail.View()
	}

	if a.statusMessage != "" {
		status := lipgloss.NewStyle().
			Foreground(lipgloss.Color("#F59E0B")).
			Render(a.statusMessage)
		view = lipgloss.JoinVertical(lipgloss.Left, view, status)
	}

	return view
}

func Run(core *beancore.Core, cfg *config.Config) error {
	app := New(core, cfg)
	p := tea.NewProgram(app, tea.WithAltScreen())
	app.program = p

	// Subscribe to file changes
	events := core.Subscribe()
	go func() {
		for range events {
			p.Send(beansChangedMsg{})
		}
	}()
	defer core.Unsubscribe(events)

	_, err := p.Run()
	return err
}
```

---

## Step 3: Update cmd/root.go with file watching

Replace `superbeans/cmd/root.go` with:

```go
package cmd

import (
	"fmt"
	"os"

	"github.com/anthropics/beans/internal/beancore"
	"github.com/anthropics/beans/internal/config"
	"github.com/anthropics/beans/superbeans/internal/tui"
	"github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
	Use:   "superbeans",
	Short: "SDLC-focused TUI for beans workflow",
	Long:  `A dedicated TUI that surfaces SDLC state at a glance—showing features grouped by status, their pipeline phase, active sessions, and review flags.`,
	RunE: func(cmd *cobra.Command, args []string) error {
		cfg, err := config.Load()
		if err != nil {
			return fmt.Errorf("failed to load config: %w", err)
		}

		core, err := beancore.New(cfg)
		if err != nil {
			return fmt.Errorf("failed to initialize beancore: %w", err)
		}

		// Start file watcher
		if err := core.StartWatching(); err != nil {
			fmt.Fprintf(os.Stderr, "Warning: file watching disabled: %v\n", err)
		}
		defer core.StopWatching()

		return tui.Run(core, cfg)
	},
}

func Execute() error {
	return rootCmd.Execute()
}

func init() {
	// Add any persistent flags here if needed
}
```

---

## Step 6: Final testing

```bash
go build -o ./superbeans-bin ./superbeans && ./superbeans-bin
```

Test:
1. Features list loads with session indicators
2. Press 'a' to attach (should fail gracefully if no session)
3. Press 'enter' to drill into feature
4. Press 'esc' to go back
5. Press 'r' to refresh
6. Edit a bean file externally - TUI should update
7. Press 'q' to quit

---

## Step 7: Commit

```bash
git add superbeans/
git commit -m "feat(superbeans): add integration and polish

- Add 'a' key to attach to tmux sessions
- Add file watching for live updates
- Add 'r' key to manually refresh
- Add periodic session status refresh (every 5s)
- Add status message feedback for errors

Refs: beans-c9yu"
```

---

## Step 8: Update mise task

Modify `.mise.toml`:

```toml
[tasks.superbeans]
description = "Build and run superbeans TUI"
run = "go build -o ./superbeans-bin ./superbeans && ./superbeans-bin"

[tasks."superbeans:build"]
description = "Build superbeans binary"
run = "go build -o ./superbeans-bin ./superbeans"
```

---

## Step 9: Final commit

```bash
git add .mise.toml
git commit -m "chore: update mise tasks for superbeans

Refs: beans-c9yu"
```

---

## Checklist

- [ ] Attach action ('a' key) implemented
- [ ] File watching wired up
- [ ] Status messages for feedback
- [ ] Periodic session refresh
- [ ] Manual refresh ('r' key)
- [ ] All navigation works
- [ ] mise tasks updated
- [ ] Committed