---
# beans-l9fx
title: 'SUPERBEANS: Add toggleable bean preview in detail view'
status: completed
type: feature
priority: normal
created_at: 2026-01-02T15:56:14Z
updated_at: 2026-01-05T15:50:03Z
---

Show markdown preview of selected bean's body in detail view. Toggle with `v`.

## Motivation

Currently, to see a bean's body content you have to:

1. Press `enter` to open in editor, or
2. Use `beans show <id>` in terminal

A preview pane lets you quickly scan bean contents (task specs, progress, notes) without leaving superbeans.

## Design

### Layout: Horizontal Split (Bottom Panel)

When preview is enabled, split the detail view with a fixed-height preview pane at the bottom (10-12 lines):

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  SUPERBEANS  →  Add User Auth                                                │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ✓ Research → ✓ Design → ✓ Plan → ◐ Implement (3/5)                          │
│                                                                              │
│  ✓    feature   Add User Auth                                    beans-abc1  │
│  ─────────────────────────────────────────────────────────────────────────── │
│  ✓    design    Design Doc: Auth System                          beans-d01   │
│▶ ●    impl      Phase 3 - Frontend                               beans-i03   │
│  ○    impl      Phase 4 - Tests                                  beans-i04   │
│                                                                              │
├────────────────────────────────── PREVIEW ───────────────────────────────────┤
│                                                                              │
│  ## Spec                                                                     │
│                                                                              │
│  Implement frontend authentication components:                               │
│  - Login form with validation                                                │
│  - Session management                                                        │
│  - Protected route wrapper                                                   │
│                                                                              │
│  ## Progress                                                                 │
│  - [x] Login form component                                                  │
│  - [x] Form validation                                                       │
│  - [ ] Session context                                                       │
│                                                                              │
╰──────────────────────────────────────────────────────────────────────────────╯
 [v]iew  [!] review  [a]ttach  [r]efresh  [y]ank  [←/esc] back  [q]uit
```

**Why horizontal split:**
- More width for markdown content (beans often have long lines)
- Simpler to implement (no column reflow needed)
- Natural reading flow (list above, content below)

**Why fixed height (not percentage):**
- Keeps the bean list usable even on smaller terminals
- Preview is scrollable, so fixed height is fine

### Keybindings

| Key | Action |
|-----|--------|
| `v` | Toggle preview on/off |
| `ctrl+d` | Scroll preview down (when visible) |
| `ctrl+u` | Scroll preview up (when visible) |
| `j`/`k` | Navigate bean list (preview follows selection) |

No focus switching needed - `j`/`k` always navigates the list, `ctrl+d`/`ctrl+u` always scroll the preview.

## Technical Approach

### Markdown Rendering

Use **glamour** (Charmbracelet) with light mode and word wrap:

```go
import "github.com/charmbracelet/glamour"

r, _ := glamour.NewTermRenderer(
    glamour.WithStylePath("light"),
    glamour.WithWordWrap(width),
)
out, _ := r.Render(content)
```

### State Management

Add to `detailModel`:

```go
type detailModel struct {
    // existing fields...

    showPreview   bool   // toggle state
    previewScroll int    // scroll offset
    previewBody   string // cached raw body text
}
```

Note: Store raw body, re-render on each `View()` call. Glamour is fast enough, and this avoids stale content if terminal width changes.

### Bean Body Loading

Fetch on-demand via GraphQL when:
- Preview is toggled on, OR
- Selection changes while preview is visible

Clear cached body when selection changes.

### Layout Calculation

```go
const previewHeight = 12 // fixed height

func (m detailModel) View() string {
    if m.showPreview {
        listHeight := m.height - previewHeight - 3 // footer + divider
        listView := m.renderList(listHeight)
        previewView := m.renderPreview(previewHeight)
        return lipgloss.JoinVertical(lipgloss.Left, listView, previewView)
    }
    return m.renderList(m.height)
}
```

## Implementation Tasks

- [x] Add glamour dependency
- [x] Add state fields to `detailModel`: `showPreview`, `previewScroll`
- [x] Add `v` key handler to toggle preview
- [x] Add `ctrl+d`/`ctrl+u` handlers for scrolling
- [x] Implement `renderPreview()` using glamour with light mode
- [x] Update `View()` to split layout with fixed preview height
- [x] Add divider styling between list and preview
- [x] Update footer: add `[v]iew` shortcut
- [x] Add tests for preview functionality

Note: Bean body loading was simplified - beans already have their `Body` field populated when loaded via the resolver, so no additional GraphQL query was needed.

## Decisions

| Question | Decision |
|----------|----------|
| Default state | Preview off |
| Split ratio | Fixed height (~12 lines) |
| Theme | Light mode (glamour) |
| Line wrapping | Soft wrap (glamour's WordWrap) |
| Toggle key | `v` (for "view") |
| Scroll keys | `ctrl+d`/`ctrl+u` only |

## Related

- [[beans-qr4m]] - Superbeans v2 thinking
- `github.com/charmbracelet/glamour` - Markdown rendering library

