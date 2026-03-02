---
name: research
description: Research the codebase and gather information for a task. Intensity is configurable (low, medium, high).
---

# Research

You are conducting research for a development task. The task ID is provided in `$ARGUMENTS` (e.g., `/dev:research my-task-id` or `/dev:research my-task-id high`).

## 1. Load task state

Parse `$ARGUMENTS` to extract the task ID and optional intensity override. Read `task_<task-id>/state.json` and `task_<task-id>/context.md`.

If the task doesn't exist, tell the user to run `/dev:setup` first.

## 2. Determine research intensity

Use the intensity from `$ARGUMENTS` if provided, otherwise fall back to `researchLevel` in state.json.

### Low
- Read the files directly relevant to the task (mentioned in context.md or obvious from the description)
- Summarize what exists, how it works, and what needs to change
- Quick and focused — stay in the main conversation

### Medium
- Use the Agent tool with subagent_type=Explore to search the codebase for relevant patterns, dependencies, and related code
- Identify architectural patterns and conventions that the implementation should follow
- Check for existing tests, types, and utilities that can be reused
- Summarize findings with file paths and line references

### High
- Launch multiple parallel research agents:
  - One to map the codebase structure and dependencies
  - One to analyze patterns and conventions
  - One to search for related prior work (PRs, docs, existing implementations)
- Cross-reference findings across agents
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
