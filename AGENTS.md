# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal Claude Code plugin repository containing the `dev` plugin — a phased development workflow for building features and fixing bugs. Skills are markdown files with YAML frontmatter, not compiled code. There is no build system, test framework, or dependencies.

## Loading the Plugin

```bash
claude --plugin-dir ./dev
```

## Repository Structure

```
dev/
  .claude-plugin/plugin.json   # Plugin manifest (name, version, author)
  agents/                      # Subagent definitions (.md files)
    codebase-explorer.md       # Research: codebase structure and dependencies
    pattern-analyzer.md        # Research/Review: patterns and conventions
    prior-work-searcher.md     # Research: related prior work and docs
    correctness-reviewer.md    # Review: logic errors, edge cases, error handling
    security-reviewer.md       # Review: security vulnerabilities, performance
    architecture-reviewer.md   # Review: architectural consistency, design
    test-runner.md             # Test: execute a single check command (haiku)
    task-adapter.md            # Persistence: dispatch task CRUD to configured backend
  task-adapters/               # Backend implementations for task persistence
    CONTRACT.md                # Adapter interface specification (5 operations)
    local.md                   # Local file adapter (task_<id>/ directories)
    linear.md                  # Linear adapter (skeleton — not yet implemented)
  skills/                      # Each subdirectory is a skill
    setup/SKILL.md             # Gather context via Q&A, output summary
    research/SKILL.md          # Explore codebase at low/medium/high depth
    plan/SKILL.md              # Generate phased implementation plan
    implement/SKILL.md         # Execute plan one phase at a time
    test/SKILL.md              # Run automated checks in parallel
    review/SKILL.md            # Code review at low/medium/high depth
    commit/SKILL.md            # Stage and commit with conventional commits
    task/SKILL.md              # Manage task persistence (create, load, save, status)
```

## Architecture

**Plugin structure**: The `dev/` directory is a Claude Code plugin. `plugin.json` defines the plugin metadata. Skills are namespaced as `/dev:<skill-name>` (e.g., `/dev:setup`, `/dev:research`).

**Skills are stateless**: Skills do not interact with task persistence directly. They operate on conversation context only — reading from what's already in the conversation and outputting results to the conversation. This makes skills composable and independent of any storage backend.

**Task management**: The `/dev:task` skill is the single entry point for all task persistence. It reads `.dev-config.json`, resolves the adapter, and spawns the `task-adapter` agent. No other skill references adapters, `.dev-config.json`, or the task-adapter agent.

**Typical workflow**:
1. `/dev:setup` — gather context via Q&A, output summary to conversation
2. `/dev:task create` — persist the gathered context (optional)
3. `/dev:research <topic>` — research the codebase, output findings
4. `/dev:task save research` — persist findings (optional)
5. `/dev:plan` — generate plan from conversation context
6. `/dev:task save plan` — persist plan (optional)
7. `/dev:implement` — execute next plan phase
8. `/dev:review` — review changes
9. `/dev:commit` — commit with conventional format

Steps 2, 4, 6 are optional — the workflow works without any persistence.

**Agents**: The `agents/` directory contains subagent definitions that skills invoke via the Agent tool. Research and review skills at medium/high intensity delegate to specialized agents for parallel analysis. The `test-runner` agent is a lightweight (haiku model) agent that executes a single check command — the `/dev:test` skill launches multiple test-runner instances in parallel. The implement skill can also delegate exploration or pattern analysis to agents when it would save context or improve results.

**Task persistence**: The `/dev:task` skill owns all adapter interaction. It reads `.dev-config.json` at the repo root to determine which backend to use (defaults to `local`). It spawns the `task-adapter` agent, which reads the adapter file and follows backend-specific instructions.

**Adapter contract**: `dev/task-adapters/CONTRACT.md` defines the formal interface that any adapter must implement — 5 operations with typed inputs/outputs, required behaviors, and error responses.

**Task adapters**: Backend implementations that the task-adapter agent follows:
- `local.md` — stores state and artifacts as files in `task_<id>/` directories at the repo root
- `linear.md` — maps tasks to Linear issues and artifacts to comments (skeleton, not yet implemented)

To add a new backend, create a new `.md` file in `task-adapters/` implementing all operations from `CONTRACT.md` and update the agent's tool list if the backend requires additional tools (e.g., MCP tools).

**Skill file format**: Each `SKILL.md` has YAML frontmatter with `description`, `disable-model-invocation: true`, and `allowed-tools` (granular per-skill tool permissions). The body is markdown instructions that Claude follows when the skill is invoked.

**Agent file format**: Each agent `.md` file has YAML frontmatter with `name`, `description`, `tools`, `model`, and `maxTurns`. The body is a system prompt defining the agent's behavior. Agent tool access is scoped narrowly (e.g., test-runner only gets `Bash`). Skills restrict which agents they can spawn via `Agent(agent-name, ...)` in `allowed-tools`.

## Conventions

- Skills have no `name` field in frontmatter — namespacing comes from the directory name under `skills/` combined with the plugin name.
- `allowed-tools` should be scoped as narrowly as possible per skill (principle of least privilege). Bash commands use glob patterns (e.g., `Bash(git diff *)`). Agent access is scoped to named agents (e.g., `Agent(test-runner)`).
- `disable-model-invocation: true` is set on all skills to prevent automatic triggering — skills must be explicitly invoked.
- Skills are stateless — they read from and output to the conversation. Only `/dev:task` interacts with persistence.
- Commits use Conventional Commits format (see `dev/skills/commit/SKILL.md`).
- Test commands combine fix + quiet flags where possible (e.g., `eslint --fix --quiet`) so a single invocation auto-fixes and reports only remaining issues.
- `.dev-config.json` at the repo root configures the task backend. If absent, defaults to `local`.
