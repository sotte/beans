---
# beans-u596
title: Improve beans prime and superpowers integration
status: completed
type: feature
priority: normal
created_at: 2026-01-05T13:42:44Z
updated_at: 2026-01-05T15:50:47Z
---

Audit and improve how beans prime plays together with superpowers skills and superbeans wrappers.

## Audit Findings

### Current State

**Three layers of integration:**
1. `beans prime` (cmd/prime.go + prompt.tmpl) - injects CLI knowledge into agent
2. User-level skills (~/.claude/commands/bean-*.md) - workflow wrappers
3. Project-level skills (skills/*.md) - superbeans-specific wrappers

**How they play together:**
- SessionStart hook runs `beans prime` → agent knows CLI commands
- User invokes `/bean-brainstorm` → skill wraps `superpowers:brainstorming`
- Skill stores output in beans instead of files
- Artifact tags (artifact:research, artifact:design, artifact:plan, artifact:impl) track phase
- TUI infers phase from artifact tags

### Issues Found

1. **Duplication**: User commands and project skills are nearly identical
   - `~/.claude/commands/bean-brainstorm.md` ≈ `skills/brainstorm/SKILL.md`
   - Only difference: naming in announcements

2. **Naming inconsistency**: Three different naming conventions
   - User commands: `bean-brainstorm`, `bean-research`
   - Project skills: `superbeans:brainstorm`, `superbeans:research`  
   - Docs/README: references both inconsistently

3. **Prime is generic**: `prompt.tmpl` teaches CLI but not the superbeans workflow
   - Mentions workflow briefly but agents still need skills for full process
   - Custom `.beans-prime.md` override exists but no example

4. **No custom prime for superbeans project**: This repo could benefit from a tailored prime

5. **Research skill inconsistency**: Uses `1_ce_research_codebase` (not a superpowers skill)
   - Other skills wrap `superpowers:*` skills
   - Research is the odd one out

6. **Missing iteration flows**: No skill for revisiting/revising previous phases

## Suggested Improvements

### 1. Create `.beans-prime.md` for superbeans project
A custom prime that:
- Teaches the full workflow (research → brainstorm → plan → execute)
- Explains artifact tags and their meaning
- Shows how to query beans by artifact type
- Includes TUI integration tips (needs-review tag, phase inference)

### 2. Consolidate skills
Either:
- A) Remove user-level bean-* commands, use only project skills
- B) Keep user-level as canonical, remove project duplicates
- C) Make user-level import/reference project skills

### 3. Standardize naming
Pick one: `bean-*` OR `superbeans:*` - not both

### 4. Add workflow examples to prime
The current prime.tmpl mentions workflow but does not show:
- How to find artifacts: `beans query ... filter: { tag: ["artifact:design"] }`
- How phases connect
- What `needs-review` tag means

### 5. Consider prime sections/modules
Instead of one monolithic prime, allow composable sections:
- Core CLI (always included)
- Workflow (optional, for superbeans users)
- GraphQL (optional, for power users)

## Questions to Answer

- Should user-level or project-level skills be canonical?
- What should a custom prime.md for superbeans look like?
- Should prime template support sections/composition?