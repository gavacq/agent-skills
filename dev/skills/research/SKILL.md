---
description: Research the codebase and gather information for a task. Intensity is configurable (low, medium, high).
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Agent(codebase-explorer, pattern-analyzer, prior-work-searcher), mcp__context7__*
---

# Research

You are conducting research for a development task. The task ID is provided in `$ARGUMENTS` (e.g., `/dev:research my-task-id` or `/dev:research my-task-id high`).

## 1. Load task state

Parse `$ARGUMENTS` to extract the task ID and optional intensity override. Read `task_<task-id>/state.json` and `task_<task-id>/context.md`.

If the task doesn't exist, tell the user to run `/dev:setup` first.

If `phases.research` is `"completed"` in state.json **and** `task_<task-id>/research.md` exists, ask the user whether they want to regenerate research (overwriting the existing file) or keep it and skip this phase. Do not proceed until the user confirms.

## 2. Determine research intensity

Use the intensity from `$ARGUMENTS` if provided, otherwise fall back to `researchLevel` in state.json.

### Low
- Read the files directly relevant to the task (mentioned in context.md or obvious from the description)
- Summarize what exists, how it works, and what needs to change
- Quick and focused — stay in the main conversation

### Medium
- Launch a `codebase-explorer` agent with the task description and scope from context.md
- The agent will map structure, trace dependencies, and identify relevant files, tests, types, and utilities
- Summarize the agent's findings with file paths and line references

### High
- Launch three agents in parallel:
  - `codebase-explorer`: Map the codebase structure and dependencies relevant to the task
  - `pattern-analyzer`: Analyze coding patterns, conventions, and style norms in the relevant areas
  - `prior-work-searcher`: Search for related prior work, documentation, and existing implementations
- Provide each agent with the task description from context.md
- Cross-reference findings across all three agents
- Produce a comprehensive research document with architecture notes, dependency maps, and implementation recommendations

## 3. Write research findings

Create or update `task_<task-id>/research.md` with:
- Summary of findings
- Key files and their roles (with paths and line numbers)
- Patterns and conventions to follow
- Dependencies and potential impacts
- Open questions or risks

## 4. Update state

Update `task_<task-id>/state.json`:
- Set `phases.research` to `"completed"`
- Set `phase` to `"research"`
- Update `updated` timestamp

## 5. Summarize and suggest next step

Tell the user what was found and suggest `/dev:plan <task-id>` as the next step.
