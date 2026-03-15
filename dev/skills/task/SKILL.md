---
description: Manage task persistence — create, load, save artifacts, update phase, and show status. Single entry point for all adapter interaction.
allowed-tools: Read, Glob, Agent(task-adapter-local), Agent(task-adapter-notion)
---

# Task

You are managing task persistence. `$ARGUMENTS` contains a subcommand and optional parameters.

## Usage

```
/dev:task create                — create task from gathered context in conversation
/dev:task load <id>             — load task into conversation
/dev:task save <type> [content] — save artifact (reads from conversation if no content)
/dev:task update <phase> <status> — update phase status
/dev:task status [id]           — show task status or list all
```

## Resolve adapter

Read `.dev-config.json` at the repo root. If it exists and has a `taskAdapter` field, use that value; otherwise default to `local`.

Map the adapter name to an agent and adapter file path:

| `taskAdapter` value | Agent | Adapter file |
|---------------------|-------|--------------|
| `local` (or absent) | `task-adapter-local` | `dev/task-adapters/local.md` |
| `notion` | `task-adapter-notion` | `dev/task-adapters/notion.md` |

If the adapter file does not exist, tell the user: "No task adapter configured. Create `.dev-config.json` with a `taskAdapter` field, or ensure `dev/task-adapters/local.md` exists."

Use the resolved agent name and adapter file path in all subcommand invocations below.

## Subcommands

### create

Create a new task from context gathered earlier in the conversation (typically via `/dev:setup`).

Expect the conversation to contain:
- Task ID (kebab-case slug)
- Tier (small or medium)
- Context markdown (description, scope, constraints, tools, commit scopes, test commands)
- Test commands (JSON object)
- Commit scopes (JSON array)
- Tools (JSON array)

If any required info is missing from the conversation, ask the user for it.

Derive `researchLevel` and `reviewLevel` from tier: small → low, medium → medium.

Spawn the task-adapter agent:

> Agent(<resolved-agent>):
> Adapter file: \<adapter path\>
> ---
> Operation: create-task
> id: \<task-id\>
> tier: \<tier\>
> researchLevel: \<level\>
> reviewLevel: \<level\>
> context: \<context markdown\>
> testCommands: \<JSON object\>
> commitScopes: \<JSON array\>
> tools: \<JSON array\>

Report the created task ID and suggest next steps.

### load

Load a task's full state and artifacts into the conversation.

> Agent(<resolved-agent>):
> Adapter file: \<adapter path\>
> ---
> Operation: load-task
> taskId: \<task-id\>

If not found, tell the user.

If found, present:
- Task state summary (phase, tier, timestamps)
- Each artifact that exists (context, research, plan, review)

The loaded data is now in the conversation for other skills to use.

### save

Save an artifact to the task. `$ARGUMENTS` format: `save <type> [content]`

- `type` is one of: `context`, `research`, `plan`, `review`
- If `content` is provided inline, use it
- If no content is provided, look for the most recent output of the corresponding type in the conversation (e.g., research findings, generated plan, review results)

If no task ID is in the conversation context, ask the user for it.

> Agent(<resolved-agent>):
> Adapter file: \<adapter path\>
> ---
> Operation: save
> taskId: \<task-id\>
> type: \<type\>
> content: \<content\>

Confirm the save.

### update

Update a task's phase status. `$ARGUMENTS` format: `update <phase> <status>`

- `phase` is one of: `setup`, `research`, `plan`, `implement`
- `status` is one of: `pending`, `in_progress`, `completed`

> Agent(<resolved-agent>):
> Adapter file: \<adapter path\>
> ---
> Operation: update-phase
> taskId: \<task-id\>
> phase: \<phase\>
> status: \<status\>

Confirm the update.

### status

Show task status. If a task ID is provided, show details. If not, list all tasks.

**With task ID:**

> Agent(<resolved-agent>):
> Adapter file: \<adapter path\>
> ---
> Operation: load-task
> taskId: \<task-id\>

Display:
- **Task ID**, **Tier**, **Current phase**
- **Phase status**: each phase with its status
- **Research/Review levels**
- **Created/Updated** timestamps
- **Plan progress**: if a plan exists, show checkbox completion (e.g., "5/12 steps completed")

**Without task ID:**

> Agent(<resolved-agent>):
> Adapter file: \<adapter path\>
> ---
> Operation: list-tasks

Display a summary table:

| Task ID | Tier | Phase | Updated |
|---------|------|-------|---------|

If no tasks exist, tell the user and suggest `/dev:setup` to gather context, then `/dev:task create` to persist.
