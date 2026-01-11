---
# beans-gg2u
title: 'SUPERBEANS: Research and document TUI best practices, add teatests'
status: completed
type: feature
priority: normal
created_at: 2026-01-03T00:16:33Z
updated_at: 2026-01-03T00:35:05Z
---

Research Bubbletea/Lipgloss/Bubbles best practices, document in docs/llm/, then identify and add missing teatests.

## Motivation

The superbeans TUI is growing in complexity. Before adding more features (like the properties modal), we should:

1. Research best practices from the Charm ecosystem
2. Document patterns for LLM consumption (in `docs/llm/`)
3. Ensure we have adequate test coverage with teatest

## Resources

- https://github.com/charmbracelet/bubbletea - The Elm Architecture for Go
- https://github.com/charmbracelet/bubbles - Common TUI components
- https://github.com/charmbracelet/lipgloss - Style definitions
- https://github.com/charmbracelet/x - Extended utilities (including teatest)

## Phase 1: Research (artifact:research)

Research tasks to be executed via subagents:

### Task 1.1: Bubbletea patterns
- Model composition (nested vs flat)
- Command patterns and batching
- Message routing between components
- Modal/overlay patterns
- Focus management

### Task 1.2: Lipgloss best practices
- Style composition and inheritance
- Responsive layouts
- Color theming
- Performance considerations

### Task 1.3: Bubbles component patterns
- When to use bubbles vs custom components
- Extending/wrapping bubbles components
- Common pitfalls

### Task 1.4: Teatest patterns
- Golden file testing
- Simulating user input
- Testing async commands
- Snapshot strategies

## Phase 2: Documentation (artifact:plan)

Create markdown files in `superbeans/docs/llm/`:

- [ ] `bubbletea-patterns.md` - Model architecture, commands, messages
- [ ] `lipgloss-styling.md` - Styles, layouts, responsive design
- [ ] `bubbles-components.md` - Using and extending bubbles
- [ ] `teatest-testing.md` - Testing strategies and examples

These docs are for LLM consumption - concise, example-heavy, pattern-focused.

## Phase 3: Implementation (artifact:impl)

### Task 3.1: Audit current test coverage
- List all TUI files and their test coverage
- Identify critical paths without tests
- Prioritize based on complexity and risk

### Task 3.2: Add teatests
Based on audit, add tests for:
- [ ] Features view navigation and rendering
- [ ] Detail view navigation and rendering
- [ ] Status picker modal
- [ ] Key bindings and shortcuts
- [ ] Session state display

## Success Criteria

- [ ] 4 documentation files in `superbeans/docs/llm/`
- [ ] All critical TUI paths have teatest coverage
- [ ] Tests pass with `mise test`
