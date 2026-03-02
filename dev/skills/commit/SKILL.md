---
description: Stage and commit changes for the current task with a well-structured commit message.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git *)
---

# Commit

You are committing changes for a development task. The task ID is in `$ARGUMENTS`.

## 1. Load task state

Read `task_<task-id>/state.json`, `task_<task-id>/context.md`, and `task_<task-id>/plan.md`.

## 2. Check readiness

- Run `git status` and `git diff` to see what's changed
- Ensure automated checks pass (lint, test, typecheck) if available
- If checks fail, report the failures and ask the user how to proceed

## 3. Stage changes

Stage the relevant files. Include:
- All code changes related to the task
- The task directory files (`task_<task-id>/`)
- Do NOT stage files that contain secrets (.env, credentials, etc.)

Use specific file paths rather than `git add -A`.

## 4. Draft commit message

Based on the task context, plan, and actual changes, draft a commit message:
- First line: concise summary under 72 chars (imperative mood)
- Blank line
- Body: explain the "why" — reference the task ID and summarize what was done
- Include `Task: <task-id>` in the body

Present the commit message to the user for approval before committing.

## 5. Commit

After user approval, create the commit.

## 6. Update state

Update `task_<task-id>/state.json`:
- Set `phases.commit` to `"completed"`
- Set `phase` to `"commit"`
- Update `updated` timestamp

## 7. Summary

Tell the user:
- The commit hash and message
- Suggest next steps: push, create PR, or start a new task
