---
description: Initialize a new task or resume an existing one. Creates a task ID, gathers context via Q&A, and persists task state to the repo.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(mkdir *), Bash(git checkout *), Bash(git branch *), Bash(git log *)
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
- **What commit scopes does this project use?** Conventional commits use an optional `(scope)` after the type (e.g., `feat(auth):`, `fix(parser):`). Check `git log --oneline -50` for existing scope usage in the project. Present any discovered scopes to the user and ask if they want to add or remove any. Common scopes are module or subsystem names (e.g., `auth`, `api`, `ui`, `db`, `config`). Store as an array — empty array if the project doesn't use scopes.
- **What automated checks are available?** Ask for the commands used for linting, formatting, testing, typechecking, and building. For each command, combine fix and quiet flags so a single invocation auto-fixes what it can and reports only remaining issues with minimal output. Guide the user:
  - ESLint: `--fix --quiet` (auto-fixes, reports only unfixable errors)
  - Prettier: `--write` (fixes in place; `--check` for check-only)
  - Vitest/Jest: `--silent` (no fix mode — just quiet output)
  - TypeScript: `--noEmit` (already minimal, no fix mode)
  - pytest: `-q` (quiet; no auto-fix)
  - ruff: `check --fix` (auto-fixes, reports remaining)
  - Build commands: vary (suggest `--quiet` or `--silent` if available)

  Store only commands the user actually has. Empty object if none. Example:
  ```json
  {
    "lint": "npx eslint . --fix --quiet",
    "format": "npx prettier --write .",
    "test": "npx vitest run --silent",
    "typecheck": "npx tsc --noEmit"
  }
  ```

## 2b. Configure sandbox (optional)

Help the user set up Claude Code sandbox configuration for the work repo. This controls what commands can access at the OS level — filesystem writes, network, and sensitive reads.

### Check existing config

Read `.claude/settings.local.json` in the work repo. If it exists and has a `sandbox` key, show the current config to the user and ask if they want to change it. If satisfied, skip to §3.

### Walk through settings

If no sandbox config exists, ask the user if they want to set one up. If yes, walk through each setting:

- **Filesystem writes** — "By default, sandboxed commands can only write to the project directory. Where else does your toolchain write?" Suggest common paths based on detected tools.

- **Sensitive reads to deny** — "Which paths should be blocked from read access?" Suggest common paths based on detected tools. Let the user add/remove.

- **Network domains** — "Which registries and APIs does this project need?" Suggest based on toolchain.

- **Local binding** — "Does this project run dev servers on localhost?" If yes, set `allowLocalBinding: true`.

- **Excluded commands** — "Any commands that don't work in sandbox? Docker is a common one." These bypass sandbox but still require permission approval. Let the user add/remove. 

- **Auto-allow** — Recommend `autoAllowBashIfSandboxed: true`. Explain: "Commands within sandbox boundaries run without prompting. Commands outside boundaries still prompt for approval."

### Write config

Present the full generated config to the user for review before writing. Write to `.claude/settings.local.json` (gitignored, local to the user).

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
  "commitScopes": [],
  "testCommands": {},
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
- Commit scopes (list discovered/configured scopes, or "None — scopes not used")
- Test commands (list each configured command, or "None configured")

## 5. Confirm and summarize

Tell the user:
- The task ID
- The task directory path
- Current phase (setup — completed)
- Next suggested action (e.g., `/dev:research <task-id>`)

Remind the user that task files are committed to the repo for tracking.
