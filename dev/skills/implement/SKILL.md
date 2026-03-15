---
description: Execute the next phase of the implementation plan from the conversation. Stops after each phase for review.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent(codebase-explorer, pattern-analyzer, test-runner)
---

# Implement

You are implementing the next phase of a development task.

## 1. Find the plan

Look in the conversation for a plan (from `/dev:plan` or `/dev:task load`). The plan should have phased sections with checkboxes.

If no plan is found in the conversation, ask the user to either:
- Run `/dev:plan` to generate one
- Run `/dev:task load <id>` to load an existing task with a plan
- Paste or describe the plan directly

## 2. Find the current phase

Parse the plan markdown. Find the first phase that has unchecked items (`- [ ]`). This is the phase to implement.

If all phases are complete, tell the user implementation is done. Suggest `/dev:review` or `/dev:commit`.

## 3. Consider delegating to agents

Before starting the current phase, review its steps and decide whether any agent would help:

- **`codebase-explorer`**: Useful when a step requires understanding unfamiliar code, tracing dependencies, or finding related files before making changes. Delegate exploration to keep the main context clean.
- **`pattern-analyzer`**: Useful when a step introduces a new module/component and you need to match existing conventions (naming, structure, error handling). Delegate pattern analysis rather than reading many files yourself.
- **`test-runner`**: Useful for running checks mid-phase without polluting context. Pass test commands explicitly or look for them in the conversation context (from `/dev:setup` or `/dev:task load`).

Only delegate when it saves context or provides better results than doing it inline. For straightforward code changes, work directly.

## 4. Implement the current phase

Work through each step in the current phase:
- Write the code changes described in each step
- Track completed steps (change `- [ ]` to `- [x]` in your working copy of the plan)

## 5. Run verification

After completing all steps in the current phase, run the "Verify" step:
- If test commands are available in the conversation context, launch a `test-runner` agent per command in parallel
- Otherwise execute any automated checks mentioned in the plan (lint, test, typecheck)
- If checks fail, fix the issues before proceeding

## 6. STOP and report

After completing one phase, **STOP**. Do not continue to the next phase automatically.

Tell the user:
- What was implemented in this phase
- Results of automated checks
- Which phase is next
- Suggest options:
  - `/dev:implement` to continue to the next phase
  - `/dev:review` to review what was just implemented
  - `/dev:task save plan` to persist the updated plan
  - Ask questions or discuss the implementation

This checkpoint is critical — the user must explicitly choose to continue.
