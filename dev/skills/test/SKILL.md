---
description: Run automated checks (lint, test, typecheck, build) in parallel subagents. Reads commands from task state or accepts explicit arguments.
disable-model-invocation: true
allowed-tools: Read, Glob, Agent(test-runner)
---

# Test

You are running automated checks for a development task. `$ARGUMENTS` may contain:

- A task ID (e.g., `/dev:test my-task-id`)
- Explicit commands (e.g., `/dev:test --lint "eslint . --fix --quiet" --test "vitest run --silent"`)
- Both (task ID with overrides)

## 1. Resolve test commands

### From task state (primary)

If a task ID is provided, read `task_<task-id>/state.json` and extract the `testCommands` object.

If no task ID is provided, scan the repo root for `task_*/state.json` files. If exactly one exists, use it. If multiple exist, list them and ask the user which task to use.

### From arguments (override or standalone)

If explicit commands are passed via flags (`--lint "cmd"`, `--test "cmd"`, `--typecheck "cmd"`, `--build "cmd"`, `--format "cmd"`), use those. If both a task ID and explicit commands are provided, the explicit commands override the corresponding keys from state.json.

### No commands found

If no test commands can be resolved, tell the user:
- No test commands are configured for this task
- Suggest running `/dev:setup` to configure them, or passing commands explicitly
- Show usage: `/dev:test <task-id>` or `/dev:test --lint "eslint . --fix --quiet" --test "vitest run --silent"`

Stop here — do not proceed without commands.

## 2. Launch parallel test-runner agents

For each entry in the resolved `testCommands` object, launch a `test-runner` agent using the Agent tool. Launch **all agents in a single message** so they run in parallel.

Pass each agent a prompt like:

```
Command name: lint
Command to run: eslint . --fix --quiet
```

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
