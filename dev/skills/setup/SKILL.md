---
name: setup
description: Initialize a new task or resume an existing one. Creates a task ID, gathers context via Q&A, and persists task state to the repo.
---

# Setup

You are initializing a development task. Follow these steps:

## 1. Check for existing task

If the user provides a task ID (e.g., `/dev:setup my-task-id`), check if `.dev/tasks/task_$ARGUMENTS/` exists.

- If it exists, read `.dev/tasks/task_$ARGUMENTS/state.json` and resume from the current phase. Summarize where things left off and ask the user what they'd like to do next.
- If it does not exist, treat `$ARGUMENTS` as context for creating a new task (see step 2).

## 2. Gather context via Q&A

Ask the user structured questions to understand the task. Use the AskUserQuestion tool to present options where possible. Gather:

- **What is the task?** Accept any format: description, markdown, Linear ticket, PR link, pasted content.
- **What tier is this?** Small (bug fix, tweak) or Medium (feature, refactor).
- **What is the scope?** Which parts of the codebase are involved?
- **What tools/MCPs are relevant?** (e.g., Context7, database access, browser, etc.)
- **Any constraints?** (e.g., no breaking changes, must support X, deadline)

## 3. Generate a task ID

Based on the Q&A, generate a short, descriptive, kebab-case task ID (e.g., `fix-auth-redirect`, `add-daily-quest-logic`). If the user provided an ID or name hint in `$ARGUMENTS`, use that as the basis. Present the proposed ID and let the user confirm or adjust.

## 4. Create task directory and files

Create the following structure:

```
.dev/tasks/task_<task-id>/
  state.json
  context.md
```

### state.json

```json
{
  "id": "<task-id>",
  "tier": "small|medium",
  "phase": "setup",
  "researchLevel": "low|medium|high",
  "reviewLevel": "low|medium|high",
  "created": "<ISO timestamp>",
  "updated": "<ISO timestamp>",
  "phases": {
    "setup": "completed",
    "research": "pending",
    "plan": "pending",
    "implement": "pending",
    "review": "pending",
    "commit": "pending"
  },
  "tools": ["list", "of", "relevant", "mcps"],
  "planFile": null
}
```

Set default research/review levels based on tier:
- **Small**: research=low, review=low
- **Medium**: research=medium, review=medium

### context.md

Write a markdown file capturing all the context gathered in the Q&A. Include:
- Task description
- Tier and scope
- Relevant tools/MCPs
- Constraints
- Any pasted content or references

## 5. Confirm and summarize

Tell the user:
- The task ID
- The task directory path
- Current phase (setup — completed)
- Next suggested action (e.g., `/dev:research <task-id>`)

Remind the user that task files are committed to the repo for tracking.
