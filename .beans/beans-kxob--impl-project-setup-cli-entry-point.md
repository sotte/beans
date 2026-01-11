---
# beans-kxob
title: 'Impl 1: Project Setup & CLI Entry Point'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-01T18:57:48Z
updated_at: 2026-01-01T19:42:34Z
parent: beans-kwuv
---

# Phase 1: Project Setup & CLI Entry Point

**Files:**
- Create: `superbeans/main.go`
- Create: `superbeans/cmd/root.go`
- Create: `superbeans/internal/tui/app.go`
- Modify: `go.mod` (if needed for workspace)
- Modify: `.mise.toml` (add build task)

---

## Step 1: Create directory structure

```bash
mkdir -p superbeans/cmd superbeans/internal/tui
```

---

## Step 2: Create main.go entry point

Create `superbeans/main.go`:

```go
package main

import (
	"os"

	"github.com/anthropics/beans/superbeans/cmd"
)

func main() {
	if err := cmd.Execute(); err != nil {
		os.Exit(1)
	}
}
```

---

## Step 3: Create cobra root command

Create `superbeans/cmd/root.go`:

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

## Step 4: Create minimal TUI app skeleton

Create `superbeans/internal/tui/app.go`:

```go
package tui

import (
	"fmt"

	"github.com/anthropics/beans/internal/beancore"
	"github.com/anthropics/beans/internal/config"
	"github.com/anthropics/beans/internal/graph"
	tea "github.com/charmbracelet/bubbletea"
)

type viewState int

const (
	viewFeatures viewState = iota
	viewDetail
)

type App struct {
	state    viewState
	resolver *graph.Resolver
	config   *config.Config
	width    int
	height   int
}

func New(core *beancore.Core, cfg *config.Config) *App {
	resolver := &graph.Resolver{Core: core}
	return &App{
		state:    viewFeatures,
		resolver: resolver,
		config:   cfg,
	}
}

func (a *App) Init() tea.Cmd {
	return nil
}

func (a *App) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	switch msg := msg.(type) {
	case tea.KeyMsg:
		switch msg.String() {
		case "q", "ctrl+c":
			return a, tea.Quit
		}
	case tea.WindowSizeMsg:
		a.width = msg.Width
		a.height = msg.Height
	}
	return a, nil
}

func (a *App) View() string {
	return fmt.Sprintf("superbeans TUI - %dx%d\n\nPress 'q' to quit", a.width, a.height)
}

func Run(core *beancore.Core, cfg *config.Config) error {
	app := New(core, cfg)
	p := tea.NewProgram(app, tea.WithAltScreen())
	_, err := p.Run()
	return err
}
```

---

## Step 5: Update .mise.toml with build task

Add to `.mise.toml`:

```toml
[tasks.superbeans]
description = "Build and run superbeans"
run = "go build -o ./superbeans-bin ./superbeans && ./superbeans-bin"
```

---

## Step 6: Verify it compiles and runs

```bash
go build -o ./superbeans-bin ./superbeans
./superbeans-bin
```

Expected: TUI opens, shows dimensions, 'q' quits cleanly.

---

## Step 7: Commit

```bash
git add superbeans/ .mise.toml
git commit -m "feat(superbeans): add project skeleton and CLI entry point

- Create superbeans/ directory with main.go and cmd/root.go
- Add minimal Bubbletea app skeleton
- Add mise task for building superbeans

Refs: beans-kxob"
```

---

## Checklist

- [ ] Directory structure created
- [ ] main.go created
- [ ] cmd/root.go with cobra command
- [ ] internal/tui/app.go with minimal App
- [ ] .mise.toml updated
- [ ] Compiles and runs
- [ ] Committed