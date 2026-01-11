---
# beans-qr4m
title: 'Superbeans v2: Agent-first workflow system redesign'
status: draft
type: feature
created_at: 2026-01-02T15:39:08Z
updated_at: 2026-01-02T15:39:08Z
---

Long-term exploration of redesigning beans/superbeans for agent-centric workflows. Captures learnings from current system and ideas for improvement.

## Context

Discussion between Stefan and Claude (2026-01-02) analyzing the current beans + superbeans + superpowers workflow and exploring what an agent-first redesign might look like.

## Current System Analysis

### Architecture

```
beans (CLI)          - Markdown-based issue tracker with GraphQL API
superbeans (TUI)     - Dashboard showing features, phases, sessions
superpowers (skills) - Claude Code skills for SDLC workflow
bean-* (commands)    - Thin wrappers connecting skills to beans
```

### What Works Well

| Component | Why it works |
|-----------|--------------|
| **Explicit SDLC phases** | Research → Design → Plan → Implement prevents diving into code without thinking |
| **Human checkpoints** | `needs-review` tag creates gates where humans validate before work proceeds |
| **Flat-file storage** | Markdown + YAML frontmatter is git-friendly, readable, survives tooling changes |
| **Session awareness** | tmux session detection shows which features have active agents |
| **Artifact tags** | Clean categorization (`artifact:research`, `artifact:design`, `artifact:plan`, `artifact:impl`) enables phase inference |
| **Worktree isolation** | Each feature gets its own branch/workspace; agents can't step on each other |
| **Bean bodies as task specs** | impl beans contain full task details; agent reads bean and knows exactly what to do |
| **Subagent-driven execution** | Fresh context per task, no pollution |

### Key Insight

> **Beans track artifacts. Skills drive workflow. Superbeans shows artifacts but not workflow.**

The disconnect: superbeans shows *bean state* (status, phase, children) but *workflow state* lives in the skills (which task in the plan? which review stage?). When an agent runs `subagent-driven-development`, it's iterating through tasks, dispatching subagents, doing review loops—none of that is visible in beans.

**However:** Stefan clarified this doesn't matter much. The phase progress (e.g., "Implement 25/26") is sufficient. The internal subagent machinery isn't important to surface.

### Actual Pain Points

1. **"When does the agent need me?"** - `needs-review` doesn't distinguish:
   - Agent asked a question (blocked, needs response NOW)
   - Agent finished artifact (needs review SOON)
   - Agent errored out (needs attention NOW)

2. **Session start ceremony** - Agent must query beans, read docs, figure out state before working. Could be streamlined.

## Agent Perspective: What Would Help

### Session Start Friction

Every time an agent starts, it has to:
1. Figure out what feature it's working on (query beans)
2. Read the parent bean, design doc, plan
3. Find which impl bean to work on
4. Understand what was already done

A `beans prime <feature-id>` command could output everything needed in one call.

### Richer Blocking States

Instead of just `needs-review`, use tags that tell you *what kind* of input is needed:

| Tag | Meaning |
|-----|---------|
| `awaiting:approval` | Work is done, needs sign-off |
| `awaiting:decision` | Blocked on specific decision |
| `awaiting:review` | Please review this artifact |
| `awaiting:unblock` | Waiting for another bean to complete |

### Structured Bean Body Sections

Convention for bean body layout:

```markdown
## Spec
[Original task spec - immutable]

## Progress
- [x] Step 1: Setup
- [x] Step 2: Core logic
- [ ] Step 3: Tests

## Notes
### 2026-01-02
- Discovered edge case X, handled with Y
- Decision: Using approach Z because...

## Handoff
Current state: Step 2 complete, Step 3 in progress
Blocker: None
Next: Write tests for edge cases in Notes
```

### Session Handoff

When ending a session, write structured handoff notes. When resuming, read these first. Currently learnings are lost between sessions.

## Design Options

### Option A: Minimal - Conventions Only

Keep beans as-is, add:
1. `beans prime <feature-id>` - one command for agent context
2. Blocking tags convention - `awaiting:*` taxonomy
3. Bean body convention - Spec/Progress/Notes/Handoff sections
4. Session log - simple append-only file

No new infrastructure. Just conventions and one aggregation command.

### Option B: Workflow-Aware Layer

Beans stay simple (artifacts with status). New component watches tmux sessions and infers state. Session log files track what's happening inside each agent. Dashboard shows both bean state AND real-time workflow state.

### Option C: Agent-First Redesign

Rethink from scratch:
- Sessions as first-class citizens
- Explicit blocking states in data model
- Structured handoff objects
- Phase artifacts with defined schemas
- Progress tracking that's queryable

## Constraints

- Stefan prefers simple systems that compose nicely
- Current workflow is working reasonably well
- Skills already do heavy lifting (discipline is in skills, not beans)
- Should build on existing infrastructure where possible

## Claude's Wish List

From my perspective as an agent:

1. **One command to prime context** - Don't make me query + read + figure out state
2. **Richer "why" when blocked** - Let me say what I need, not just "needs attention"
3. **Handoff persistence** - What I learned this session should help next session
4. **Progress visibility** - For long tasks, show I'm on step 7/10

## Open Questions

1. Should beans gain new fields, or should state live elsewhere?
2. How much workflow visibility is actually needed vs nice-to-have?
3. Is the current system 80% good enough, needing only small additions?
4. Would a `beans prime` command solve most of the session start friction?

## Next Steps

- [ ] Implement idle/working indicator first (see beans-lb0j)
- [ ] Experiment with `beans prime` command
- [ ] Try structured bean body conventions
- [ ] Evaluate if more is needed after living with these changes

## Related

- [[beans-lb0j]] - Idle/working indicator (near-term)
- `/home/stefan/dotfiles/dotfiles/claude/docs/SUPERBEANS.md` - Current workflow docs