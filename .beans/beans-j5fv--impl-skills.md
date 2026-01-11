---
# beans-j5fv
title: 'Impl: Skills'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:47:20Z
updated_at: 2026-01-05T00:53:35Z
parent: beans-js3n
blocking:
    - beans-t4iz
---

# Phase 2: Skills

## Files

- Create: `skills/research/SKILL.md`
- Create: `skills/brainstorm/SKILL.md`
- Create: `skills/plan/SKILL.md`
- Create: `skills/execute/SKILL.md`

## Steps

**Step 1: Create skills directory structure**

```bash
mkdir -p skills/research skills/brainstorm skills/plan skills/execute
```

**Step 2: Create research skill**

Create `skills/research/SKILL.md`:

```markdown
---
name: research
description: Research codebase context for a feature, storing findings in a research bean. Use before brainstorming when you need to understand existing code.
---

# Bean Research

## Overview

Wrapper around `1_ce_research_codebase` that stores output in beans instead of `_spec/research/` files.

**Announce at start:** "I'm using superbeans:research to gather context for this feature."

## Bean Structure & Workflow

` ` `
● Feature (type: feature)                        ← reading or creating
  ├── Research (task, artifact:research)         ← CREATING THIS
  ├── Design Doc (task, artifact:design)
  ├── Implementation Plan (task, artifact:plan)
  └── Impl phases (task, artifact:impl)...
` ` `

## Input

**Option A: Existing epic/feature bean**
- User provides bean ID via $ARGUMENTS
- Read the bean to understand what to research

**Option B: No bean yet**
- Ask user to describe what they want to research
- Create feature bean: `beans create "<title>" -t feature -s draft -d "<description>"`

## Process

**Follow the `1_ce_research_codebase` approach:**

1. Read any directly mentioned files first
2. Analyze and decompose the research question
3. Spawn parallel sub-agents for comprehensive research:
   - **codebase-locator** - find WHERE files and components live
   - **codebase-analyzer** - understand HOW specific code works
   - **codebase-pattern-finder** - find examples of existing patterns
4. Wait for all sub-agents to complete
5. Synthesize findings into a coherent research document

**Key difference from `1_ce_research_codebase`:** Output goes to bean body, not `_spec/research/`.

## Output

1. Create research bean with `artifact:research` tag:
   ` ` `bash
   beans create "Research: <topic>" -t task -s in-progress --parent <parent-id> --tag artifact:research
   ` ` `

2. Write research content to bean body using this structure:
   ` ` `markdown
   # Research: <topic>

   ## Research Question
   [Original query]

   ## Summary
   [High-level findings]

   ## Detailed Findings

   ### [Component/Area 1]
   - Description ([file.ext:line])
   - How it connects to other components

   ### [Component/Area 2]
   ...

   ## Code References
   - `path/to/file.py:123` - Description
   - `another/file.ts:45-67` - Description

   ## Architecture Notes
   [Patterns, conventions, design decisions found]

   ## Open Questions
   [Areas needing further investigation]
   ` ` `

3. Mark research bean as needing review:
   ` ` `bash
   beans update <research-bean-id> --tag needs-review
   ` ` `

4. Report:
   ` ` `
   Research complete:
   - Parent: <parent-id>
   - Research: <research-bean-id> (needs-review)

   Next: Review findings, then run /superbeans:brainstorm-cmd <parent-id>
   ` ` `
```

**Step 3: Create brainstorm skill**

Create `skills/brainstorm/SKILL.md`:

```markdown
---
name: brainstorm
description: Brainstorm an idea into a design, storing result in a design bean. Use when starting a new feature with beans workflow.
---

# Bean Brainstorm

## Overview

Wrapper around `superpowers:brainstorming` that stores output in beans instead of files.

**Announce at start:** "I'm using superbeans:brainstorm to develop the design for this feature."

## Bean Structure & Workflow

` ` `
● Feature (type: feature)                        ← reading or creating
  ├── Research (task, artifact:research)         (optional, created by superbeans:research)
  ├── Design Doc (task, artifact:design)         ← CREATING THIS
  ├── Implementation Plan (task, artifact:plan)
  └── Impl phases (task, artifact:impl)...
` ` `

## Input

**Option A: Existing epic/feature bean**
- User provides bean ID via $ARGUMENTS
- Read the bean to understand the rough idea

**Option B: No bean yet**
- Ask user to describe the idea
- Create feature bean: `beans create "<title>" -t feature -s in-progress -d "<description>"`

## Process

**Execute the `superpowers:brainstorming` skill exactly.**

## Output (instead of writing to docs/plans/)

1. Create design bean with `artifact:design` tag:
   ` ` `bash
   beans create "Design Doc: <feature-name>" -t task -s in-progress --parent <parent-id> --tag artifact:design
   ` ` `

2. Write design content to bean body

3. Update parent bean body with architecture overview

4. Mark design bean as needing review:
   ` ` `bash
   beans update <design-bean-id> --tag needs-review
   ` ` `

5. Report:
   ` ` `
   Design complete:
   - Parent: <parent-id> (updated with overview)
   - Design Doc: <design-bean-id> (needs-review)

   Next: Review the design, then run /superbeans:plan-cmd <design-bean-id>
   ` ` `
```

**Step 4: Create plan skill**

Create `skills/plan/SKILL.md`:

```markdown
---
name: plan
description: Create implementation plan from design bean, generating task beans for each phase/task.
---

# Bean Write Plan

## Overview

Wrapper around `superpowers:writing-plans` that stores output in beans instead of files.

**Announce at start:** "I'm using superbeans:plan to create the implementation plan."

## Bean Structure & Workflow

` ` `
● Feature (type: feature)
  ├── Research (task, artifact:research)         ✓ completed (if used)
  ├── Design Doc (task, artifact:design)         ← reading
  ├── Implementation Plan (task, artifact:plan)  ← CREATING THIS
  ├── Impl: Phase 1 (task, artifact:impl)        ← CREATING THESE
  ├── Impl: Phase 2 (task, artifact:impl)        ← CREATING THESE
  └── Impl: Phase N (task, artifact:impl)        ← CREATING THESE
` ` `

## Input

- User provides design bean ID via $ARGUMENTS
- Read the design bean to understand what to implement
- Identify the parent (epic/feature) from the design bean

## Process

**Execute the `superpowers:writing-plans` skill exactly.**

## Output (instead of writing to docs/plans/)

1. Create Implementation Plan bean with `artifact:plan` tag:
   ` ` `bash
   beans create "Implementation Plan: <feature-name>" -t task -s in-progress --parent <parent-id> --tag artifact:plan
   ` ` `
   - Write plan overview to bean body (architecture, tech stack, sequence)

2. For each phase in the plan, create implementation bean with `artifact:impl` tag:
   ` ` `bash
   beans create "Impl: <phase-name>" -t task -s todo --parent <parent-id> --tag artifact:impl
   ` ` `
   - Write full phase content to bean body (files, steps, code, commit message)

3. Update Implementation Plan bean body with links to impl phase beans

4. Mark design bean as completed:
   ` ` `bash
   beans update <design-bean-id> -s completed
   ` ` `

5. Mark plan bean as needing review:
   ` ` `bash
   beans update <plan-bean-id> --tag needs-review
   ` ` `

6. Report:
   ` ` `
   Plan complete:
   - Parent: <parent-id>
   - Design Doc: <design-bean-id> (completed)
   - Implementation Plan: <plan-bean-id> (needs-review)
   - Phases created: <phase-bean-1>, <phase-bean-2>, ...

   Next: Review the plan, then run /superbeans:execute-cmd <parent-id>
   ` ` `
```

**Step 5: Create execute skill**

Create `skills/execute/SKILL.md`:

```markdown
---
name: execute
description: Execute implementation tasks from beans, updating status as you progress.
---

# Bean Execute Plan

## Overview

Wrapper around `superpowers:executing-plans` that reads from and updates beans.

**Announce at start:** "I'm using superbeans:execute to implement the tasks."

## Bean Structure & Workflow

` ` `
● Feature (type: feature)
  ├── Research (task, artifact:research)         ✓ completed (if used)
  ├── Design Doc (task, artifact:design)         ✓ completed
  ├── Implementation Plan (task, artifact:plan)  ✓ completed
  ├── Impl: Phase 1 (task, artifact:impl)        ← EXECUTING THESE
  ├── Impl: Phase 2 (task, artifact:impl)        ← EXECUTING THESE
  └── Impl: Phase N (task, artifact:impl)        ← EXECUTING THESE
` ` `

## Input

- User provides parent bean ID via $ARGUMENTS
- Read the parent to get overview and list of implementation task/phase beans

## Execution Approach

Before starting, ask:

**"Use subagent-driven development? (fresh subagent per task, code review between tasks)"**

**If yes:**
- **REQUIRED SUB-SKILL:** Use `superpowers:subagent-driven-development`
- Dispatches fresh subagent for each task bean
- Code review between tasks

**If no:**
- **REQUIRED SUB-SKILL:** Use `superpowers:executing-plans`
- Execute in batches of 3 tasks
- Report between batches for review

Both approaches: read task beans, execute, mark completed, commit.

## Process

Execute using chosen approach. Each implementation task bean IS a plan task—the bean body contains the full task details (files, steps, code, commit message) written by superbeans:plan.

Before starting tasks:
1. Mark parent as in-progress:
   ` ` `bash
   beans update <parent-id> -s in-progress
   ` ` `

For each task/phase:
1. Read the implementation bean body for full task details
2. Mark task as in-progress:
   ` ` `bash
   beans update <task-bean-id> -s in-progress
   ` ` `
3. Execute the task
4. After implementation and review, mark bean as completed:
   ` ` `bash
   beans update <task-bean-id> -s completed
   ` ` `
5. Commit the changes

## Observations During Implementation

File beans for inconsistencies, tech debt, or improvements observed during implementation:
` ` `bash
beans create "<observation>" -t task -s draft -p low -d "<details>"
` ` `

Don't derail current work—capture and move on.

## After All Tasks Complete

1. Report completion to user - do NOT auto-complete the parent feature (human decides when feature is done)

2. Use `superpowers:finishing-a-development-branch` for final verification.
```

**Step 6: Commit**

```bash
git add skills/
git commit -m "feat: add workflow skills

- research: codebase research with bean output
- brainstorm: design ideation with bean output  
- plan: implementation planning with bean tasks
- execute: task execution with bean status tracking

All skills updated with superbeans: namespace references.

Refs: beans-js3n"
```