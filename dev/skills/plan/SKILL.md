---
description: Generate a phased implementation plan with checkboxes from context and research in the conversation.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Plan

You are creating an implementation plan for a development task.

## 1. Gather context from conversation

Look in the conversation for:
- Task context (from `/dev:setup` or `/dev:task load`)
- Research findings (from `/dev:research`)
- Any other relevant discussion

If no context or research is available in the conversation, ask the user to provide:
- A description of what needs to be done
- Relevant files or areas of the codebase
- Any constraints

## 2. Generate the plan

Based on the context and research, generate a phased implementation plan. Each phase should be:
- A self-contained unit of work that can be implemented, tested, and reviewed independently
- Small enough to complete in one session
- Ordered so that earlier phases don't depend on later ones

### Plan format

```markdown
# Plan: <task description or id>

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

## 3. Present for review

Show the user the full plan. Ask them to:
- Confirm it looks good
- Request changes to any phase
- Add, remove, or reorder phases
- Adjust scope

If the user requests changes, update the plan and re-present.

Do NOT proceed to implementation until the user explicitly approves.

## 4. Suggest next steps

Once the plan is approved, suggest:
- `/dev:implement` to start implementing the first phase
- `/dev:task save plan` to persist the plan (if a task is active in the conversation)
