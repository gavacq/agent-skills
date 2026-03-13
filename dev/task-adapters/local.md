# Local File Adapter

This adapter implements the contract defined in `CONTRACT.md`.

Store task state and artifacts as files in a `task_<id>/` directory at the repository root.

## File structure

```
task_<id>/
  state.json      # Task state (phase, config, timestamps)
  context.md      # Task description, scope, constraints
  research.md     # Research findings (optional)
  plan.md         # Implementation plan (optional)
  review.md       # Review findings (optional)
```

## Operations

### create-task

1. Write `task_<id>/state.json`:

```json
{
  "id": "<id>",
  "tier": "<tier>",
  "phase": "setup",
  "researchLevel": "<researchLevel>",
  "reviewLevel": "<reviewLevel>",
  "created": "<ISO timestamp>",
  "updated": "<ISO timestamp>",
  "phases": {
    "setup": "completed",
    "research": "pending",
    "plan": "pending",
    "implement": "pending"
  },
  "tools": <tools array>,
  "commitScopes": <commitScopes array>,
  "testCommands": <testCommands object>,
  "planFile": null
}
```

2. Write `task_<id>/context.md` with the provided context content.

### load-task

1. Read `task_<id>/state.json`. If the file does not exist, return "not found".
2. Read each artifact file if it exists (use Glob to check):
   - `task_<id>/context.md`
   - `task_<id>/research.md`
   - `task_<id>/plan.md`
   - `task_<id>/review.md`
3. Return a structured response with `state` (parsed JSON) and each artifact as a string (or null if the file doesn't exist).

### save

1. Write the content to `task_<id>/<type>.md`.
2. If `type` is `plan`, also update `task_<id>/state.json` field `planFile` to `"task_<id>/plan.md"`.

### update-phase

1. Read `task_<id>/state.json`.
2. Set `phase` to the given phase name.
3. Set `phases.<phase>` to the given status.
4. Set `updated` to the current ISO timestamp.
5. If `extras` is provided, merge each key-value pair into the state object.
6. Write the updated state back to `task_<id>/state.json`.

### list-tasks

1. Use Glob to find all `task_*/state.json` files at the repository root.
2. Read each `state.json`.
3. Return an array of `{ id, tier, phase, updated }` for each task.
4. If no task directories exist, return an empty array.
