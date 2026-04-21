# Workflow

How to work with Claude and agents on this project.

---

## Session Start Checklist

Before starting any work (~1 minute):

1. **Read `plan.md`** -- check "Current focus" and "Handoff Notes."
2. **Spot-check relevant docs** -- if relying on a design or architecture doc, verify key claims against the actual code.
3. **Pick your agent** -- see agent table in CLAUDE.md or use `claude --agent <name>`.

## Agent Workflow

For each piece of work, pick one primary agent. Full specs live in `.claude/agents/`.

### Standard feature workflow

```
1. Planner Agent   --> scope feature, write UX goals, update plan.md
2. Domain Agent    --> data models, services, business logic
3. UI Agent        --> views, styling, interactions
4. Testing Agent   --> tests for new behavior
5. Docs Agent      --> history.md, plan.md, commit
```

Use `/clear` between agent phases to keep context small.

### Quick recipes

**Bugfix:**
1. Testing Agent: write regression test reproducing the bug
2. Domain/UI Agent: fix until test passes
3. Docs Agent: update plan.md and commit

**Small UI tweak:**
1. UI Agent with plan.md + design-language.md + the view file
2. Docs Agent if the change is user-visible

**Feedback iteration (user corrects implementation):**
1. UI/Domain Agent: apply the corrected approach
2. Docs Agent: document feedback in feedback.md, update history.md
