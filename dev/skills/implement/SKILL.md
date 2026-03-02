---
description: Execute the next phase of the implementation plan. Stops after each phase for review. Runs automated checks between phases.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent
---

# Implement

You are implementing the next phase of a development task. The task ID is in `$ARGUMENTS`.

## 1. Load task state

Read `task_<task-id>/state.json` and `task_<task-id>/plan.md`.

If no plan exists, tell the user to run `/dev:plan` first.

If `phases.implement` is `"completed"` in state.json **and** all checkboxes in `plan.md` are checked, ask the user whether they want to re-run implementation or skip this phase. Do not proceed until the user confirms.

## 2. Find the current phase

Parse the plan markdown. Find the first phase that has unchecked items (`- [ ]`). This is the phase to implement.

If all phases are complete, mark `phases.implement` as `"completed"` in state.json and tell the user implementation is done. Suggest `/dev:review <task-id>` or `/dev:commit <task-id>` as next steps.

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

Update `task_<task-id>/state.json`:
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
