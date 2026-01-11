---
# beans-1uxh
title: 'Design Doc: Custom Beans UI'
status: completed
type: task
priority: normal
tags:
    - artifact:design
created_at: 2026-01-01T18:42:31Z
updated_at: 2026-01-01T19:04:32Z
parent: beans-kwuv
---

# Design Doc: Custom Beans UI for SUPERBEANS Workflow

**Status**: Draft v1.0
**Author**: Stefan + Claude Code
**Date**: 2026-01-01

## 1. Problem Statement

The current beans TUI is a general-purpose tree view. It's helpful for browsing all beans but not optimized for the SUPERBEANS SDLC workflow where we need:

1. **Quick visibility** into which features need review
2. **Session awareness** - which features have active Claude sessions
3. **Phase tracking** - where is each feature in the pipeline (research → design → plan → implement)

**Goal**: A custom TUI that surfaces SDLC state at a glance and enables quick action (attach to session, review, drill down).

## 2. Entry Point & Scope

### CLI

- **Command**: `superbeans` (new CLI, separate from `beans`)
- **Location**: Dedicated folder in beans repo (e.g., `superbeans/`)
- **Relationship**: `beans` continues to open the existing tree TUI; `superbeans` opens the new SDLC-focused TUI

### Filtering

- **Shown**: Features with status `in-progress`, `todo`, `completed`
- **Hidden**: Features with status `draft` (not actionable yet), `scrapped`

## 3. Views

### View 1: Features Overview (Home)

The main entry point. Shows all features grouped by state.

```
╭──────────────────────────────────────────────────────────────────────────────╮
│  SUPERBEANS                                                    [F]eatures   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  IN PROGRESS                                                                 │
│  ●  Add User Auth                  ⚑ ◐ impl 3/5     ● live      org-abc1   │
│  ●  Payment Integration            ⚑ ◐ design       ● live      org-abc2   │
│  ●  Dashboard Revamp                 ◐ impl 1/3     ● live      org-abc3   │
│  ●  API Refactor                     ◐ research     ○ none      org-abc4   │
│                                                                              │
│  TODO                                                                        │
│  ○  Email Notifications              ○ idea         ○ none      org-abc5   │
│  ○  Admin Panel                      ○ idea         ○ none      org-abc6   │
│                                                                              │
│  DONE (recent)                                                               │
│  ✓  Login Fixes                      ✓ complete     ○ none      org-abc7   │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯
 [enter] drill down  [a]ttach  [q]uit
```

**Columns**: Status | Title | Review flag + Phase | Session | ID

**Grouping**: In Progress → Todo → Done (sorted by state)

### View 2: Feature Detail

Drill down into a specific feature to see its children (research, design, plan, impl phases).

```
╭──────────────────────────────────────────────────────────────────────────────╮
│  ← org-abc1  Add User Auth                                ● live  [a]ttach  │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ✓ Research → ✓ Design → ✓ Plan → ◐ Implement (3/5)                         │
│                                                                              │
│  ●     Add User Auth                               feature        org-abc1  │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ✓     Research: Auth Patterns                     research       org-r01   │
│  ✓     Design Doc: Auth System                     design         org-d01   │
│  ✓     Implementation Plan                         plan           org-p01   │
│  ✓     Phase 1 - Database                          impl 1/5       org-i01   │
│  ✓     Phase 2 - API                               impl 2/5       org-i02   │
│▌ ●  ⚑  Phase 3 - Frontend                          impl 3/5       org-i03   │
│  ○     Phase 4 - Tests                             impl 4/5       org-i04   │
│  ○     Phase 5 - Docs                              impl 5/5       org-i05   │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯
 [enter] view bean  [a]ttach  [esc] back  [q]uit
```

**Elements**:
- Pipeline summary at top (visual "where are we")
- Parent feature row
- Separator
- Children list with status, title, phase tag, ID

**Columns**: Status | Review flag | Title | Phase/Artifact tag | ID

## 4. Visual Language

### Status Indicators

| Symbol | Meaning | Used for |
|--------|---------|----------|
| `✓` | Done/Complete | Bean status, phase complete |
| `●` | In progress | Bean status, active work |
| `○` | Todo/Not started | Bean status, pending work |
| `⚑` | Needs review | Flag for human attention |

### Session Indicators

| Symbol | Meaning |
|--------|---------|
| `● live` | tmux session exists for this feature |
| `○ none` | No session |

### Phase Progress

| Symbol | Meaning | Example |
|--------|---------|---------|
| `◐` | Phase in progress | `◐ impl 3/5` |
| `✓` | Phase complete | `✓ design` |
| `○` | Phase not started | `○ idea` |

### Phase Labels

| Label | Meaning |
|-------|---------|
| `idea` | No children yet |
| `research` | Research artifact in progress |
| `design` | Design artifact in progress |
| `plan` | Plan artifact in progress |
| `impl N/M` | Implementing, N of M phases done |
| `complete` | All children done |

### Pipeline Summary

```
✓ Research → ✓ Design → ✓ Plan → ◐ Implement (3/5)
```

Uses `✓`, `○`, `◐` symbols with `→` connectors.

## 5. Keyboard Controls

| Key | Action |
|-----|--------|
| `j/k` or `↑/↓` | Navigate |
| `enter` | Drill in / view detail |
| `esc` | Back |
| `a` | Attach to session |
| `q` | Quit |

## 6. Data Requirements

The UI needs to query:

1. **Features**: All beans of type `feature` with their status
2. **Children**: For each feature, its child beans with artifact tags
3. **Sessions**: tmux session list to match against feature bean IDs
4. **Needs-review**: Beans with `needs-review` tag

### Phase Inference Logic

```
if no children → "idea"
if any child has artifact:research AND status=in-progress → "research"
if any child has artifact:design AND status=in-progress → "design"
if any child has artifact:plan AND status=in-progress → "plan"
if any child has artifact:impl →
    count completed impl / total impl → "impl N/M"
if all children completed → "complete"
```

### Needs-review Bubble Up

A feature shows `⚑` if ANY of its children have the `needs-review` tag.

## 7. Technical Notes

- TUI framework: TBD (could use same as current beans TUI, or Textual/Rich for Python)
- Session detection: `tmux list-sessions` and match against `<bean-id>-*` pattern
- Attach action: `tmux attach -t <session-name>` or create if missing via `beans-worktree`

## 8. Future Considerations (v2+)

- Bean body viewer (full content of design docs, plans)
- Activity feed / recent changes
- Notifications for `needs-review` (desktop, sound)
- Dynamic session status (running vs idle vs waiting)

## Related

- [[beans-kwuv]] - Parent feature
- [[org-jreb]] - SUPERBEANS epic
