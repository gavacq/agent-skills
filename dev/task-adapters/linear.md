# Linear Adapter

This adapter implements the contract defined in `CONTRACT.md`.

Store task state and artifacts in Linear issues and comments.

> **Status:** Skeleton — not yet implemented. This file documents the intended mapping from the persistence API to Linear's data model.

## Prerequisites

- Linear MCP server must be configured and accessible
- `.dev-config.json` must include Linear-specific config:

```json
{
  "taskAdapter": "linear",
  "linear": {
    "teamId": "<Linear team ID>",
    "projectId": "<Linear project ID (optional)>"
  }
}
```

## Data model mapping

| Persistence concept | Linear equivalent |
|---|---|
| Task ID | Issue identifier (e.g., `ENG-123`) |
| Task state (phase, tier) | Issue labels + custom fields |
| Context | Issue description |
| Research artifact | Comment with `[research]` tag |
| Plan artifact | Comment with `[plan]` tag |
| Review artifact | Comment with `[review]` tag |
| Phase status | Issue status (mapped via workflow states) |
| Test commands | Stored in issue description footer or a pinned comment |
| Commit scopes | Stored in issue description footer or a pinned comment |

## Tools required

The task-adapter agent will need these MCP tools when using this adapter:

- `mcp__claude_ai_Linear__get_issue` — load task state and context
- `mcp__claude_ai_Linear__save_issue` — create/update task
- `mcp__claude_ai_Linear__list_comments` — load artifacts
- `mcp__claude_ai_Linear__save_comment` — save artifacts
- `mcp__claude_ai_Linear__list_issues` — list tasks
- `mcp__claude_ai_Linear__get_issue_status` — map phase to workflow state
- `mcp__claude_ai_Linear__list_issue_statuses` — discover available states
- `mcp__claude_ai_Linear__get_team` — resolve team context

## Operations

### create-task

1. Create a Linear issue using `save_issue`:
   - Title: task ID or description summary
   - Description: the context markdown, with a metadata footer containing `testCommands` and `commitScopes` as a fenced JSON block
   - Team: from config `teamId`
   - Project: from config `projectId` (if set)
   - Labels: `tier:<tier>`, `phase:setup`
2. Return the created issue identifier.

### load-task

1. Fetch the issue using `get_issue` with the task ID.
2. Parse the description for context and metadata footer.
3. Fetch all comments using `list_comments`.
4. Find comments tagged `[research]`, `[plan]`, `[review]` — use the most recent of each.
5. Reconstruct state from issue labels/status.
6. Return structured response matching the API contract.

### save

1. Search existing comments for one matching `[<type>]` tag.
2. If found, update it. If not, create a new comment with the tag prefix:
   ```
   [<type>]
   <content>
   ```
3. If `type` is `plan`, also update the issue description metadata to include the plan reference.

### update-phase

1. Update the issue's status to the workflow state that maps to the given phase + status.
2. Update labels: remove old `phase:*` label, add `phase:<phase>`.
3. Add a brief comment noting the phase transition (for audit trail).

### list-tasks

1. Use `list_issues` filtered by team and project (from config).
2. For each issue, extract id, tier (from label), phase (from status/label), and updated timestamp.
3. Return the array.
