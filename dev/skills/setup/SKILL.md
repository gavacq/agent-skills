---
description: Initialize a new task or resume an existing one. Creates a task ID, gathers context via Q&A, and persists task state to the repo.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(mkdir *), Bash(git checkout *), Bash(git branch *)
---

# Setup

You are initializing a development task. Follow these steps:

## 1. Resolve task ID

Try to determine the task ID in this order:

1. **Explicit argument**: If the user provides a task ID via `$ARGUMENTS`, use that.
2. **Branch name**: Run `git branch --show-current`. If the branch name is not `main` or `master`, use it as the task ID (it should already be kebab-case).
3. **Build from context**: If neither of the above yields a task ID, proceed to Q&A (step 2) and generate one from the gathered context.

Once you have a candidate task ID, check if `task_<task-id>/` exists:

- If it exists, read `task_<task-id>/state.json` and resume from the current phase. Summarize where things left off and ask the user what they'd like to do next.
- If it does not exist, continue to step 2 to create a new task.

## 2. Gather context via Q&A

Ask the user structured questions to understand the task. Use the AskUserQuestion tool to present options where possible. Gather:

- **What is the task?** Accept any format: description, markdown, Linear ticket, PR link, pasted content.
- **What tier is this?** Small (bug fix, tweak) or Medium (feature, refactor).
- **What is the scope?** Which parts of the codebase are involved?
- **What tools/MCPs are relevant?** (e.g., Context7, database access, browser, etc.)
- **Any constraints?** (e.g., no breaking changes, must support X, deadline)

## 3. Confirm task ID

If the task ID was resolved from the branch name or argument, present it for confirmation. If it was not resolved yet, generate a short, descriptive, kebab-case ID from the Q&A context (e.g., `fix-auth-redirect`, `add-daily-quest-logic`). Present the proposed ID and let the user confirm or adjust.

## 4. Create task directory and files

Create the following structure:

```
task_<task-id>/
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
    "implement": "pending"
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
