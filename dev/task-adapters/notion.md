# Notion Adapter

This adapter implements the contract defined in `CONTRACT.md`.

Store task state as a Notion database page and artifacts as child sub-pages.

> **Note on MCP tool names:** The tool names below assume a Notion MCP server configured with the prefix `mcp__notion__`. Verify that these match the actual tool names in your `.mcp.json` server configuration. Common alternatives include `notion_` prefixed tools without the MCP namespace.

## Prerequisites

Read `.dev-config.json` to obtain:
- `notion.databaseId` — the Notion database ID where tasks are stored

Required database properties (the user must create these manually in Notion):

| Property  | Type       | Usage |
|-----------|------------|-------|
| `Title`   | title      | Task ID (kebab-case) |
| `Tier`    | select     | `small` or `medium` |
| `Phase`   | select     | Current phase name |
| `Updated` | date       | ISO timestamp of last update |
| `Created` | date       | ISO timestamp of creation |
| `Phases`  | rich_text  | JSON: `{"setup":"completed","research":"pending",...}` |
| `Config`  | rich_text  | JSON: `{researchLevel, reviewLevel, tools, commitScopes, testCommands, planFile}` |

## Artifact storage

Each artifact (`context`, `research`, `plan`, `review`) is stored as a child sub-page of the task page. The child page title is the capitalized artifact name (e.g., "Context", "Research"). Content is stored as paragraph blocks. Notion blocks have a 2000-character limit — split content at 2000-char boundaries across multiple paragraph blocks.

## Operations

### create-task

1. Read `databaseId` from `.dev-config.json` → `notion.databaseId`.
2. Call `notion_create_page` with parent `databaseId` and properties:
   - `Title`: task id
   - `Tier`: tier
   - `Phase`: `"setup"`
   - `Created`: current ISO timestamp
   - `Updated`: current ISO timestamp
   - `Phases`: JSON string `{"setup":"completed","research":"pending","plan":"pending","implement":"pending"}`
   - `Config`: JSON string `{"researchLevel":"<researchLevel>","reviewLevel":"<reviewLevel>","tools":<tools>,"commitScopes":<commitScopes>,"testCommands":<testCommands>,"planFile":null}`
3. From the response, note the new page's `id` (the task page ID).
4. Call `notion_append_block_children` on the task page to create a child page block titled `"Context"`. Note the child page block's `id`.
5. Split the context content into chunks of ≤2000 characters. Call `notion_append_block_children` on the child page to append each chunk as a paragraph block.
6. Return `{ "id": "<task-id>" }`.

### load-task

1. Call `notion_query_database` on `databaseId` with a filter: `Title` equals `taskId`. If results are empty, return `NOT_FOUND`.
2. From the first result, parse:
   - `Phases` rich_text → JSON → phases object
   - `Config` rich_text → JSON → config object (researchLevel, reviewLevel, tools, commitScopes, testCommands, planFile)
   - `Phase` select → current phase string
   - `Tier` select → tier string
   - `Created` date → created timestamp
   - `Updated` date → updated timestamp
3. Reconstruct the state object:
   ```json
   {
     "id": "<taskId>",
     "tier": "<tier>",
     "phase": "<phase>",
     "phases": <phases>,
     "researchLevel": "<researchLevel>",
     "reviewLevel": "<reviewLevel>",
     "created": "<created>",
     "updated": "<updated>",
     "tools": <tools>,
     "commitScopes": <commitScopes>,
     "testCommands": <testCommands>,
     "planFile": <planFile>
   }
   ```
4. Call `notion_retrieve_block_children` on the task page to list its child blocks. For each child page block, note its title and id.
5. For each artifact type (`context`, `research`, `plan`, `review`):
   - If a child page exists with the matching capitalized title, call `notion_retrieve_block_children` on that child page and concatenate all paragraph block text → artifact string.
   - Otherwise, set the artifact to `null`.
6. Return:
   ```json
   {
     "state": <state object>,
     "context": "<string or null>",
     "research": "<string or null>",
     "plan": "<string or null>",
     "review": "<string or null>"
   }
   ```

### save

1. Call `notion_query_database` on `databaseId` filtering by Title=taskId. If empty, return `NOT_FOUND`.
2. Note the task page id.
3. Call `notion_retrieve_block_children` on the task page to find a child page block with title matching the capitalized artifact type (e.g., `"Research"` for type `research`).
4. If the child page exists:
   a. Call `notion_retrieve_block_children` on the child page to get its blocks.
   b. Delete each existing block by calling `notion_delete_block` for each block id.
   c. Split the new content into chunks of ≤2000 characters. Call `notion_append_block_children` on the child page with each chunk as a paragraph block.
5. If the child page does not exist:
   a. Call `notion_append_block_children` on the task page to create a new child page block titled with the capitalized artifact name. Note the new child page id.
   b. Split the content into chunks of ≤2000 characters. Call `notion_append_block_children` on the child page with each chunk as a paragraph block.
6. If `type == "plan"`:
   a. Retrieve the current `Config` rich_text from the task page (re-query if needed). Parse the JSON.
   b. Set `planFile` to the child page's Notion page id.
   c. Call `notion_update_page` on the task page to update the `Config` property with the new JSON.
7. Return `OK`.

### update-phase

1. Call `notion_query_database` on `databaseId` filtering by Title=taskId. If empty, return `NOT_FOUND`.
2. Note the task page id.
3. Retrieve the current `Phases` and `Config` rich_text properties from the page result. Parse both as JSON.
4. Set `phases[phase] = status` in the phases object.
5. If `extras` is provided, merge each key-value pair into the config object.
6. Call `notion_update_page` on the task page with:
   - `Phase`: phase (select)
   - `Updated`: current ISO timestamp (date)
   - `Phases`: updated phases JSON (rich_text)
   - `Config`: updated config JSON (rich_text) — include even if unchanged
7. Return `OK`.

### list-tasks

1. Call `notion_query_database` on `databaseId` with no filter. Paginate if the response indicates more results (use `start_cursor`).
2. For each page result, extract:
   - `Title` rich_text/title → task id string
   - `Tier` select → tier string
   - `Phase` select → phase string
   - `Updated` date → updated timestamp
3. Return an array of `{ "id": "...", "tier": "...", "phase": "...", "updated": "..." }`.
4. If no pages exist, return an empty array `[]`.
