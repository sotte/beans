---
# beans-vdoo
title: 'Implementation Plan: Superbeans Plugin'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
    - needs-review
created_at: 2026-01-04T23:47:06Z
updated_at: 2026-01-05T00:55:38Z
parent: beans-js3n
---

# Superbeans Plugin Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Convert superbeans repo into a Claude Code plugin with workflow skills, commands, and hooks.

**Architecture:** Plugin structure at repo root with skills/, commands/, hooks/ directories. Thin command wrappers invoke full skill definitions. Hook auto-detects tmux and jq.

**Tech Stack:** Claude Code plugin system, Markdown skills/commands, Bash hooks.

---

## Implementation Phases

| Phase | Bean | Description | Status |
|-------|------|-------------|--------|
| 1 | beans-f132 | Plugin manifest and structure | todo |
| 2 | beans-j5fv | Skills (research, brainstorm, plan, execute) | todo |
| 3 | beans-9z3m | Commands (thin wrappers with $ARGUMENTS) | todo |
| 4 | beans-n054 | Hooks (tmux state tracking, jq check) | todo |
| 5 | beans-gbf1 | Documentation (README update) | todo |
| 6 | beans-t4iz | Test and verify | todo (blocked) |

## Dependencies

- Phases 1-5 are independent and can run in parallel
- Phase 6 (test) is blocked by phases 1-5

## Fixes Applied During Review

1. ✅ Skills now contain full transformed content with updated references
2. ✅ Internal skill references updated to use `superbeans:*` and `/superbeans:*-cmd`
3. ✅ Commands now pass `$ARGUMENTS` to skills
4. ✅ Hook checks for both tmux AND jq before running
5. ✅ Blocking relationships set (phases 1-5 block phase 6)
6. ✅ Documented that `.claude/` is for local dev, separate from plugin

## Commit Strategy

One commit per phase for clear history.