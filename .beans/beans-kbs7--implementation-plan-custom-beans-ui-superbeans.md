---
# beans-kbs7
title: 'Implementation Plan: Custom Beans UI (superbeans)'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
created_at: 2026-01-01T18:55:27Z
updated_at: 2026-01-01T20:18:23Z
parent: beans-kwuv
---

# Implementation Plan: Custom Beans UI (superbeans)

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a dedicated TUI (`superbeans`) that surfaces SDLC state at a glance—showing features grouped by status, their pipeline phase, active sessions, and review flags.

**Architecture:** New binary (`superbeans/`) sharing the existing `internal/` packages. Two views: Features Overview (home) and Feature Detail. Uses Bubbletea with same patterns as existing TUI. Adds phase inference logic and tmux session detection.

**Tech Stack:** Go, Bubbletea, Lipgloss, existing `internal/graph` resolver, `internal/ui` utilities

---

## Implementation Phases

| Phase | Title | Bean |
|-------|-------|------|
| 1 | Project Setup & CLI Entry Point | [[beans-kxob]] |
| 2 | Data Layer: Phase Inference & Session Detection | [[beans-ndib]] |
| 3 | Features Overview View | [[beans-t6ja]] |
| 4 | Feature Detail View | [[beans-rhd1]] |
| 5 | Integration & Polish | [[beans-c9yu]] |

---

## Technical Decisions

### Directory Structure
```
superbeans/
├── main.go              # Entry point
├── cmd/
│   └── root.go          # Cobra root command
└── internal/
    └── tui/
        ├── app.go       # Root App model
        ├── features.go  # Features Overview view
        ├── detail.go    # Feature Detail view
        ├── phase.go     # Phase inference logic
        ├── session.go   # tmux session detection
        └── styles.go    # superbeans-specific styles
```

### Reusing Existing Code
- `internal/graph.Resolver` - GraphQL queries/mutations
- `internal/beancore.Core` - Bean loading and file watching
- `internal/ui` - Colors, `RenderBeanRow()`, responsive columns
- `internal/config` - Configuration loading

### New Code Required
1. **Phase inference** (`phase.go`): Determine feature phase from children's artifact tags
2. **Session detection** (`session.go`): Parse `tmux list-sessions` output
3. **Features view** (`features.go`): Grouped list with phase/session columns
4. **Detail view** (`detail.go`): Pipeline summary + children list

### Key Data Structures

```go
// Phase represents where a feature is in the SDLC pipeline
type Phase struct {
    Name     string  // "idea", "research", "design", "plan", "impl", "complete"
    Progress string  // "3/5" for impl phase, empty otherwise
    Symbol   string  // "○", "◐", "✓"
}

// FeatureItem wraps a bean with computed SDLC state
type FeatureItem struct {
    Bean        *bean.Bean
    Phase       Phase
    HasSession  bool
    SessionName string
    NeedsReview bool  // Any child has needs-review tag
}
```

### Phase Inference Algorithm
```
if no children → Phase{Name: "idea", Symbol: "○"}
if any child has artifact:research AND status=in-progress → Phase{Name: "research", Symbol: "◐"}
if any child has artifact:design AND status=in-progress → Phase{Name: "design", Symbol: "◐"}
if any child has artifact:plan AND status=in-progress → Phase{Name: "plan", Symbol: "◐"}
if any child has artifact:impl →
    completed := count where status=completed
    total := count impl children
    if completed == total → Phase{Name: "complete", Symbol: "✓"}
    else → Phase{Name: "impl", Progress: fmt.Sprintf("%d/%d", completed, total), Symbol: "◐"}
if all children completed → Phase{Name: "complete", Symbol: "✓"}
```

### Session Detection
```go
func DetectSessions() (map[string]string, error) {
    // Run: tmux list-sessions -F "#{session_name}"
    // Parse output, match against bean ID patterns
    // Return map[beanID]sessionName
}
```

---

## Related
- [[beans-kwuv]] - Parent feature
- [[beans-1uxh]] - Design doc
