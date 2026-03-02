---
name: implement
description: Execute the next phase of the implementation plan. Stops after each phase for review. Runs automated checks between phases.
---

# Implement

You are implementing the next phase of a development task. The task ID is in `$ARGUMENTS`.

## 1. Load task state

Read `.dev/tasks/task_<task-id>/state.json` and `.dev/tasks/task_<task-id>/plan.md`.

If no plan exists, tell the user to run `/dev:plan` first.

## 2. Find the current phase

Parse the plan markdown. Find the first phase that has unchecked items (`- [ ]`). This is the phase to implement.

If all phases are complete, tell the user implementation is done and suggest `/dev:review <task-id>`.

## 3. Implement the current phase

Work through each step in the current phase:
- Write the code changes described in each step
- After completing a step, update the plan file: change `- [ ]` to `- [x]` for that step
- Save the plan file after each step so progress is tracked

## 4. Run verification

After completing all steps in the current phase, run the "Verify" step:
- Execute any automated checks mentioned (lint, test, typecheck)
- If checks fail, fix the issues before proceeding
- If the project has a standard check command (e.g., `npm test`, `make check`), run it

## 5. Update state

Update `.dev/tasks/task_<task-id>/state.json`:
- Set `phase` to `"implement"`
- Set `phases.implement` to `"in_progress"`
- Update `updated` timestamp

## 6. STOP and report

After completing one phase, **STOP**. Do not continue to the next phase automatically.

Tell the user:
- What was implemented in this phase
- Results of automated checks
- Which phase is next
- Suggest options:
  - `/dev:implement <task-id>` to continue to the next phase
  - `/dev:review <task-id>` to review what was just implemented
  - Ask questions or discuss the implementation

This checkpoint is critical — the user must explicitly choose to continue.
