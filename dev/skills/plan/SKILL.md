---
description: Generate a phased implementation plan with checkboxes. The plan is saved to the repo and editable before execution.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Plan

You are creating an implementation plan for a development task. The task ID is in `$ARGUMENTS`.

## 1. Load task state

Read `task_<task-id>/state.json`, `task_<task-id>/context.md`, and `task_<task-id>/research.md`.

If research hasn't been completed yet, ask the user if they want to skip research or run `/dev:research` first.

## 2. Generate the plan

Based on the context and research, generate a phased implementation plan. Each phase should be:
- A self-contained unit of work that can be implemented, tested, and reviewed independently
- Small enough to complete in one session
- Ordered so that earlier phases don't depend on later ones

### Plan format

Write the plan as a markdown file with this structure:

```markdown
# Plan: <task-id>

## Summary
<1-2 sentence description of what this plan accomplishes>

## Phase 1: <phase title>
- [ ] Step 1 description
- [ ] Step 2 description
- [ ] Verify: <what to check>

## Phase 2: <phase title>
- [ ] Step 1 description
- [ ] Step 2 description
- [ ] Verify: <what to check>

...
```

Each phase should end with a "Verify" step describing what automated checks or manual validation to run.

## 3. Save the plan

Write the plan to `task_<task-id>/plan.md`.

## 4. Update state

Update `task_<task-id>/state.json`:
- Set `phases.plan` to `"completed"`
- Set `phase` to `"plan"`
- Set `planFile` to `task_<task-id>/plan.md`
- Update `updated` timestamp

## 5. Present for review

Show the user the full plan. Ask them to:
- Confirm it looks good
- Request changes to any phase
- Add, remove, or reorder phases
- Adjust scope

If the user requests changes, update the plan file and re-present. Do NOT proceed to implementation until the user explicitly approves.

Suggest `/dev:implement <task-id>` as the next step once approved.
