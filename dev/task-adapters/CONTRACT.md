# Task Adapter Contract

Any task adapter MUST implement all five operations defined below. Adapters receive structured prompts from the `task-adapter` agent and return structured responses.

## Data types

### Artifact types
`context`, `research`, `plan`, `review`

### Phase names
`setup`, `research`, `plan`, `implement`

### Status values
`pending`, `in_progress`, `completed`

## Operations

### create-task

Create a new task with initial state and context.

**Input:**
- `id` (string, required) — kebab-case task identifier
- `tier` (string, required) — `"small"` or `"medium"`
- `researchLevel` (string, required) — `"low"`, `"medium"`, or `"high"`
- `reviewLevel` (string, required) — `"low"`, `"medium"`, or `"high"`
- `context` (string, required) — markdown with task description, scope, constraints
- `testCommands` (object, required) — map of check names to commands (may be `{}`)
- `commitScopes` (array, required) — scope strings (may be `[]`)
- `tools` (array, required) — relevant MCP/tool names (may be `[]`)

**Output:** `{ id: "<task-id>" }`

**Required behavior:**
- Initialize all phases to `pending` except `setup` which is `completed`
- Set `created` and `updated` to current ISO timestamp
- Store the context as the `context` artifact

### load-task

Load the full task state and all existing artifacts.

**Input:**
- `taskId` (string, required) — task identifier

**Output (success):**
```json
{
  "state": { "id", "tier", "phase", "phases", "researchLevel", "reviewLevel", "created", "updated", "tools", "commitScopes", "testCommands", "planFile" },
  "context": "<markdown string or null>",
  "research": "<markdown string or null>",
  "plan": "<markdown string or null>",
  "review": "<markdown string or null>"
}
```

**Output (not found):** `NOT_FOUND`

### save

Save or update a task artifact.

**Input:**
- `taskId` (string, required) — task identifier
- `type` (string, required) — one of the artifact types
- `content` (string, required) — artifact content

**Output:** `OK`

**Required behavior:**
- Overwrite any existing artifact of the same type
- If `type` is `plan`, also update the task state to reference the plan location

### update-phase

Update the current phase status in task state.

**Input:**
- `taskId` (string, required) — task identifier
- `phase` (string, required) — one of the phase names
- `status` (string, required) — one of the status values
- `extras` (object, optional) — additional state fields to merge

**Output:** `OK`

**Required behavior:**
- MUST update the task's `updated` timestamp to the current ISO timestamp
- MUST set the top-level `phase` field to the given phase name
- MUST set the phase-specific status (e.g., `phases.<phase>`) to the given status
- If `extras` is provided, merge each key-value pair into the state

### list-tasks

List all tasks with summary info.

**Input:** (none)

**Output:**
```json
[{ "id": "...", "tier": "...", "phase": "...", "updated": "..." }, ...]
```

Returns an empty array if no tasks exist.

## Error responses

- `NOT_FOUND` — task does not exist (for `load-task`, `save`, `update-phase`)
- `ADAPTER_ERROR: <message>` — unexpected error during operation

## Compound operations

When multiple operations are sent in a single prompt (separated by `---`), execute them sequentially in order. Return results for each operation. Later operations may depend on earlier ones succeeding.
