---
# beans-9z3m
title: 'Impl: Commands'
status: completed
type: task
priority: normal
tags:
    - artifact:impl
created_at: 2026-01-04T23:47:22Z
updated_at: 2026-01-05T00:54:15Z
parent: beans-js3n
blocking:
    - beans-t4iz
---

# Phase 3: Commands

## Files

- Create: `commands/research-cmd.md`
- Create: `commands/brainstorm-cmd.md`
- Create: `commands/plan-cmd.md`
- Create: `commands/execute-cmd.md`

## Steps

**Step 1: Create commands directory**

```bash
mkdir -p commands
```

**Step 2: Create research-cmd**

Create `commands/research-cmd.md`:

```markdown
---
description: "Research codebase context for a feature, storing findings in a research bean."
---

Invoke the superbeans:research skill and follow it exactly as presented to you.

ARGUMENTS: $ARGUMENTS
```

**Step 3: Create brainstorm-cmd**

Create `commands/brainstorm-cmd.md`:

```markdown
---
description: "Brainstorm an idea into a design, storing result in a design bean."
---

Invoke the superbeans:brainstorm skill and follow it exactly as presented to you.

ARGUMENTS: $ARGUMENTS
```

**Step 4: Create plan-cmd**

Create `commands/plan-cmd.md`:

```markdown
---
description: "Create implementation plan from design bean, generating task beans for each phase."
---

Invoke the superbeans:plan skill and follow it exactly as presented to you.

ARGUMENTS: $ARGUMENTS
```

**Step 5: Create execute-cmd**

Create `commands/execute-cmd.md`:

```markdown
---
description: "Execute implementation tasks from beans, updating status as you progress."
---

Invoke the superbeans:execute skill and follow it exactly as presented to you.

ARGUMENTS: $ARGUMENTS
```

**Step 6: Commit**

```bash
git add commands/
git commit -m "feat: add command wrappers for skills

- research-cmd: invokes superbeans:research
- brainstorm-cmd: invokes superbeans:brainstorm
- plan-cmd: invokes superbeans:plan
- execute-cmd: invokes superbeans:execute

All commands pass $ARGUMENTS to skills.

Refs: beans-js3n"
```