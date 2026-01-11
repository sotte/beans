---
# beans-v6e4
title: 'Implementation Plan: Work indicator ask state'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
    - needs-review
created_at: 2026-01-05T00:50:52Z
updated_at: 2026-01-05T03:01:45Z
parent: beans-1xgc
---

# Work Indicator Ask State Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a fourth session state `SessionAsk` that displays "? ask" when Claude calls `AskUserQuestion`, providing visual feedback that user input is needed.

**Architecture:** The tmux hook script detects `AskUserQuestion` tool calls and sets `@claude_state` to `idle:ask`. The TUI parses this value to render a distinct "? ask" indicator in amber color. The `Stop` event resets state to `idle`.

**Tech Stack:** Go (lipgloss for styling), Bash (hook script), tmux session options

---

## Implementation Phases

1. **[[beans-pgdd]]** - Add SessionAsk constant and parsing logic
2. **[[beans-et6o]]** - Add SessionAsk style and rendering
3. **[[beans-bbj9]]** - Update documentation
