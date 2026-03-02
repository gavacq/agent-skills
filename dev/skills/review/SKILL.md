---
name: review
description: Review code changes for the current task. Intensity is configurable (low, medium, high).
---

# Review

You are reviewing code changes for a development task. The task ID and optional intensity are in `$ARGUMENTS` (e.g., `/dev:review my-task-id` or `/dev:review my-task-id high`).

## 1. Load task state

Read `.dev/tasks/task_<task-id>/state.json` and `.dev/tasks/task_<task-id>/plan.md`.

If the task doesn't exist, tell the user to run `/dev:setup` first.

## 2. Determine review intensity

Use the intensity from `$ARGUMENTS` if provided, otherwise fall back to `reviewLevel` in state.json.

### Low
- Run automated checks (lint, test, typecheck) if available
- Quick scan of the diff — check for obvious issues
- Confirm changes align with the plan

### Medium
- Run automated checks
- Review each changed file for:
  - Correctness — does the code do what the plan says?
  - Edge cases — are there missing error handling or boundary conditions?
  - Style — does it follow the codebase conventions found during research?
- Summarize findings with specific file:line references

### High
- Run automated checks
- Launch parallel review agents:
  - One for correctness and logic review
  - One for architecture alignment and pattern consistency
  - One for security and performance concerns
- Cross-reference findings
- Produce a comprehensive review document

## 3. Write review findings

Create or update `.dev/tasks/task_<task-id>/review.md` with:
- Summary (pass/fail/needs-changes)
- Issues found (with file:line references and severity)
- Suggestions for improvement
- Automated check results

## 4. Update state

Update `.dev/tasks/task_<task-id>/state.json`:
- Set `phases.review` to `"completed"` (or `"in_progress"` if issues need fixing)
- Set `phase` to `"review"`
- Update `updated` timestamp

## 5. Present findings

Show the user the review results. If issues were found:
- List them clearly with severity
- Suggest fixes
- Ask the user how to proceed (fix and re-review, accept as-is, etc.)

If the review passes, suggest `/dev:commit <task-id>`.
