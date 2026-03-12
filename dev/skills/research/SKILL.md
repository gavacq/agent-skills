---
description: Research the codebase and gather information for a task. Intensity is configurable (low, medium, high).
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Agent(codebase-explorer, pattern-analyzer, prior-work-searcher), mcp__context7__*
---

# Research

You are conducting research for a development task. `$ARGUMENTS` may contain a task ID, intensity, and/or a freeform research scope description.

Examples:
- `/dev:research my-task-id`
- `/dev:research my-task-id high`
- `/dev:research how does the auth middleware work`
- `/dev:research high where are API routes defined`

## 1. Load task state (optional)

Parse `$ARGUMENTS` to extract:
- **Task ID**: a short slug that matches an existing `task_*/` directory (e.g. `my-task-id`)
- **Intensity**: one of `low`, `medium`, `high`
- **Scope**: any remaining text is treated as the research topic or question

If a task ID is found, read `task_<task-id>/state.json` and `task_<task-id>/context.md`.

If no task ID is recognizable (no matching `task_*/` directory exists), proceed in **taskless mode**: use the scope from `$ARGUMENTS` as the research topic. Do NOT stop or ask the user to run `/dev:setup`.

If `phases.research` is `"completed"` in state.json **and** `task_<task-id>/research.md` exists, ask the user whether they want to regenerate research (overwriting the existing file) or keep it and skip this phase. Do not proceed until the user confirms.

## 2. Determine research intensity

Use the intensity from `$ARGUMENTS` if provided, otherwise fall back to `researchLevel` in state.json. If neither is available, default to `medium`.

Use the task description from `context.md` as the research topic when a task exists. In taskless mode, use the scope from `$ARGUMENTS`.

### Low
- Read the files directly relevant to the topic
- Summarize what exists, how it works, and what needs to change
- Quick and focused — stay in the main conversation

### Medium
- Launch a `codebase-explorer` agent with the research topic
- The agent will map structure, trace dependencies, and identify relevant files, tests, types, and utilities
- Summarize the agent's findings with file paths and line references

### High
- Launch three agents in parallel:
  - `codebase-explorer`: Map the codebase structure and dependencies relevant to the topic
  - `pattern-analyzer`: Analyze coding patterns, conventions, and style norms in the relevant areas
  - `prior-work-searcher`: Search for related prior work, documentation, and existing implementations
- Cross-reference findings across all three agents
- Produce a comprehensive research document with architecture notes, dependency maps, and implementation recommendations

## 3. Write research findings

If a task exists, create or update `task_<task-id>/research.md`. In taskless mode, present findings directly without writing to disk.

Either way, include:
- Summary of findings
- Key files and their roles (with paths and line numbers)
- Patterns and conventions to follow
- Dependencies and potential impacts
- Open questions or risks

## 4. Update state

If a task exists, update `task_<task-id>/state.json`:
- Set `phases.research` to `"completed"`
- Set `phase` to `"research"`
- Update `updated` timestamp

Skip this step in taskless mode.

## 5. Summarize and suggest next step

Tell the user what was found. If a task exists, suggest `/dev:plan <task-id>` as the next step.
