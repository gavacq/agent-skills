---
description: Run automated checks (lint, test, typecheck, build) in parallel subagents. Reads commands from explicit arguments or project files.
allowed-tools: Read, Glob, Grep, Agent(test-runner)
---

# Test

You are running automated checks. `$ARGUMENTS` may contain explicit commands.

Examples:
- `/dev:test`
- `/dev:test --lint "eslint . --fix --quiet" --test "vitest run --silent"`

## 1. Resolve test commands

Resolution order — use the first source that yields commands:

### 1a. Explicit arguments (highest priority)

If explicit commands are passed via flags (`--lint "cmd"`, `--test "cmd"`, `--typecheck "cmd"`, `--build "cmd"`, `--format "cmd"`), add them to the command set. These always override matching keys from any other source.

### 1b. Conversation context

If no explicit commands are provided, check the conversation for test commands from a previous `/dev:setup` or `/dev:task load`. Use `testCommands` if available.

### 1c. Infer from project files (fallback)

If no commands are available, infer them by reading:

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
- Suggest passing commands explicitly
- Show usage: `/dev:test --lint "eslint . --fix --quiet" --test "vitest run --silent"`

Stop here — do not proceed without commands.

## 2. Launch parallel test-runner agents

For each entry in the resolved commands, launch a `test-runner` agent using the Agent tool. Launch **all agents in a single message** so they run in parallel.

Pass each agent a prompt like:

```
Command name: lint
Command to run: eslint . --fix --quiet
```

If the command fails, return the **complete** error output — do not truncate. The main agent needs the full errors to report to the user.

## 3. Collect and summarize results

After all agents complete, produce a summary:

### Test Results

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
