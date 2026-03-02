---
description: Execute the next phase of the implementation plan. Stops after each phase for review. Runs automated checks between phases.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent(codebase-explorer, pattern-analyzer, test-runner)
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

## 3. Consider delegating to agents

Before starting the current phase, review its steps and decide whether any agent would help:

- **`codebase-explorer`**: Useful when a step requires understanding unfamiliar code, tracing dependencies, or finding related files before making changes. Delegate exploration to keep the main context clean.
- **`pattern-analyzer`**: Useful when a step introduces a new module/component and you need to match existing conventions (naming, structure, error handling). Delegate pattern analysis rather than reading many files yourself.
- **`test-runner`**: Useful for running checks mid-phase without polluting context. Read `testCommands` from state.json if available.

Only delegate when it saves context or provides better results than doing it inline. For straightforward code changes, work directly.

## 4. Implement the current phase

Work through each step in the current phase:
- Write the code changes described in each step
- After completing a step, update the plan file: change `- [ ]` to `- [x]` for that step
- Save the plan file after each step so progress is tracked

## 5. Run verification

After completing all steps in the current phase, run the "Verify" step:
- If `testCommands` exists in state.json, launch a `test-runner` agent per command in parallel
- Otherwise execute any automated checks mentioned in the plan (lint, test, typecheck)
- If checks fail, fix the issues before proceeding

## 6. Update state

Update `task_<task-id>/state.json`:
- Set `phase` to `"implement"`
- Set `phases.implement` to `"in_progress"`
- Update `updated` timestamp

## 7. STOP and report

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
