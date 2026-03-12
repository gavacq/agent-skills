---
description: Run automated checks (lint, test, typecheck, build) in parallel subagents. Reads commands from task state or accepts explicit arguments.
allowed-tools: Read, Glob, Grep, Agent(test-runner)
---

# Test

You are running automated checks for a development task. `$ARGUMENTS` may contain:

- A task ID (e.g., `/dev:test my-task-id`)
- Explicit commands (e.g., `/dev:test --lint "eslint . --fix --quiet" --test "vitest run --silent"`)
- Both (task ID with overrides)

## 1. Resolve test commands

Resolution order — use the first source that yields commands:

### 1a. Explicit arguments (highest priority)

If explicit commands are passed via flags (`--lint "cmd"`, `--test "cmd"`, `--typecheck "cmd"`, `--build "cmd"`, `--format "cmd"`), add them to the command set. These always override matching keys from any other source.

### 1b. Task state

If a task ID is provided, read `task_<task-id>/state.json` and extract the `testCommands` object.

If no task ID is provided, scan the repo root for `task_*/state.json` files. If exactly one exists, use it. If multiple exist, list them and ask the user which task to use.

### 1c. Infer from project files (fallback)

If no task state is found (or `testCommands` is empty), infer commands by reading:

1. `AGENTS.md` (or `CLAUDE.md`) at the repo root — look for documented test/lint/typecheck commands in code blocks or lists
2. `package.json` — inspect the `scripts` object for keys matching: `lint`, `test`, `typecheck`, `type-check`, `build`, `format`, `check`

When building inferred commands, **always prefer low-output variants**:
- Append `--quiet` or `--silent` flags where the tool supports them
- Prefer `eslint --fix --quiet` over `eslint`
- Prefer `vitest run --silent` over `vitest run`
- Prefer `tsc --noEmit` over `tsc`
- Skip `build` commands unless the user explicitly requested them (they are slow and rarely needed for review)

Tell the user which commands were inferred and from which source.

### 1d. No commands found

If no commands can be resolved from any source, tell the user:
- No test commands could be found
- Suggest adding a `testCommands` field via `/dev:setup`, or passing commands explicitly
- Show usage: `/dev:test <task-id>` or `/dev:test --lint "eslint . --fix --quiet" --test "vitest run --silent"`

Stop here — do not proceed without commands.

## 2. Launch parallel test-runner agents

For each entry in the resolved `testCommands` object, launch a `test-runner` agent using the Agent tool. Launch **all agents in a single message** so they run in parallel.

Pass each agent a prompt like:

```
Command name: lint
Command to run: eslint . --fix --quiet
```

If the command fails, return the **complete** error output — do not truncate. The main agent needs the full errors to report to the user.

## 3. Collect and summarize results

After all agents complete, produce a summary:

### Test Results: <task-id or "standalone">

| Check | Status |
|-------|--------|
| lint | PASS |
| test | FAIL |
| typecheck | PASS |

**Overall: PASS or FAIL** (N of M checks passed)

### Failures

If any checks failed, include the error output from each failing agent under a subheading:

#### <check-name>
<error details from the test-runner agent>

If all checks pass, just show: **All checks passed.**

## 4. No state update

This skill does NOT update state.json. It is a standalone utility that can run at any time. Other skills (implement, review, commit) may run automated checks independently — this skill is for explicit, parallel execution.
