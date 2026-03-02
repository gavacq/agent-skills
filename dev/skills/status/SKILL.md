---
name: status
description: Show the current state of a task or list all tasks.
---

# Status

You are reporting on task status. `$ARGUMENTS` may contain a task ID or be empty.

## If a task ID is provided

Read `.dev/tasks/task_<task-id>/state.json` and display:

- **Task ID**: the ID
- **Tier**: small/medium
- **Current phase**: which phase is active
- **Phase status**: show each phase with its status (pending/in_progress/completed)
- **Research level**: low/medium/high
- **Review level**: low/medium/high
- **Created/Updated**: timestamps
- **Plan**: if a plan exists, show a brief summary and checkbox progress (e.g., "5/12 steps completed")

## If no task ID is provided

List all tasks by scanning `.dev/tasks/` directories. For each task, read its state.json and display a summary table:

| Task ID | Tier | Phase | Progress | Updated |
|---------|------|-------|----------|---------|

If no tasks exist, tell the user and suggest `/dev:setup` to create one.
