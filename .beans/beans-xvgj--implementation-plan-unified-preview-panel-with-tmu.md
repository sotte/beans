---
# beans-xvgj
title: 'Implementation Plan: Unified Preview Panel with Tmux Mode'
status: completed
type: task
priority: normal
tags:
    - artifact:plan
    - needs-review
created_at: 2026-01-05T21:43:11Z
updated_at: 2026-01-06T01:16:19Z
parent: beans-rmtd
---

# Unified Preview Panel with Tmux Mode

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Extend the existing preview panel to cycle through three modes (off → body → tmux → off) with the `v` key.

**Architecture:** Replace `showPreview bool` with `PreviewMode` enum. Add `CapturePane()` to session.go for tmux pane capture. Extend both detail.go and features.go to render tmux preview. Piggyback on existing 1-second session poll for live updates.

**Tech Stack:** Go, Bubbletea TUI framework, tmux CLI

---

## Implementation Phases

1. **[[beans-og1t]]** Phase 1: PreviewMode enum and cycling
2. **[[beans-p7j0]]** Phase 2: CapturePane function  
3. **[[beans-f3ig]]** Phase 3: Detail view tmux preview
4. **[[beans-ipdp]]** Phase 4: Features list preview panel
5. **[[beans-9hn1]]** Phase 5: Live tmux updates
