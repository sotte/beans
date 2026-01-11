---
# beans-bu4u
title: 'Implementation Plan: Bubbletea best practices for superbeans TUI'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
    - needs-review
created_at: 2026-01-04T23:37:49Z
updated_at: 2026-01-04T23:59:14Z
parent: beans-ld6n
---

# Bubbletea Best Practices Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add comprehensive tests for TUI Update() and keyboard interactions following documented best practices.

**Architecture:** Table-driven unit tests for each model's Update() method. Test fixtures shared across test files. CI-compatible via ASCII color profile.

**Tech Stack:** Go testing, testify/assert, lipgloss, termenv

---

## Critical Audit Finding

After thorough review, the superbeans TUI **already follows Bubbletea best practices**:
- Flat model composition with state machine (`viewFeatures`, `viewDetail`, `viewPropPicker`)
- Message-driven async I/O (no blocking in Update)
- Custom message types for component communication
- Proper use of `tea.Batch()`, `tea.Every()`, `tea.ExecProcess()`
- Package-level styles in `styles.go`
- CI-compatible test init with `termenv.Ascii`

**Primary gap:** Testing coverage for Update() and keyboard interactions.

---

## Implementation Phases

### Phase 1: Setup - Add `.gitattributes` for golden files
Create `.gitattributes` with golden file line ending protection.

### Phase 2: Test Fixtures - Create shared test helpers
Create `app_test.go` with test fixtures and helper functions.

### Phase 3: App Update Tests - Test root model navigation
Add tests for quit, view transitions, modal lifecycle, WindowSizeMsg broadcast.

### Phase 4: Features Model Tests - Test list cursor navigation
Create `features_test.go` with cursor movement and selection tests.

### Phase 5: PropPicker Tests - Test property selection shortcuts
Create `proppicker_test.go` with keyboard shortcut tests.

### Phase 6: Detail Model Tests - Test detail view navigation
Create `detail_test.go` with cursor and selection tests.

---

## Files to Modify

| File | Action |
|------|--------|
| `.gitattributes` | Create |
| `superbeans/internal/tui/app_test.go` | Create |
| `superbeans/internal/tui/features_test.go` | Create |
| `superbeans/internal/tui/proppicker_test.go` | Create |
| `superbeans/internal/tui/detail_test.go` | Create |

---

## Implementation Beans

- beans-g0z1: Phase 1 - .gitattributes setup
- beans-m59f: Phase 2 - Test fixtures and helpers
- beans-lzid: Phase 3 - App Update() tests
- beans-dkzu: Phase 4 - Features model tests
- beans-d5ue: Phase 5 - PropPicker tests
- beans-51jr: Phase 6 - Detail model tests
