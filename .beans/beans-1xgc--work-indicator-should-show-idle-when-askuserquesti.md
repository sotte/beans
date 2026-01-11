---
# beans-1xgc
title: Work indicator should show idle when AskUserQuestion is triggered
status: completed
type: feature
priority: normal
created_at: 2026-01-04T23:16:33Z
updated_at: 2026-01-05T03:08:46Z
---

The work indicator currently shows 'working' when AskUserQuestion is triggered. It should switch to an "ask" indicator since the user needs to take action.

## Motivation

When Claude asks a question, the ball is in the user's court. The indicator should reflect that we're waiting on the user, not actively working.

## Architecture Overview

Add a fourth session state `SessionAsk` that is set when Claude calls `AskUserQuestion`. The hook detects the tool name and sets `idle:ask` in tmux, which the TUI parses and renders as `? ask` in amber.

**State flow:**
- Any tool → `● work` (green)
- AskUserQuestion → `? ask` (amber)
- Stop → `○ idle` (amber)
- No session → `○ none` (dim)

## Tasks

- [x] Identify where session state is determined
- [x] Detect when AskUserQuestion tool is triggered
- [x] Design the ask state (see design doc beans-om3k)
- [ ] Implement TUI changes
- [ ] Update hook script example in docs
- [ ] Update visual-language.md
- [ ] Test the behavior