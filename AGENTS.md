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
  skills/                      # Each subdirectory is a skill
    setup/SKILL.md             # Phase: initialize task, gather context
    research/SKILL.md          # Phase: explore codebase at low/medium/high depth
    plan/SKILL.md              # Phase: generate phased implementation plan
    implement/SKILL.md         # Phase: execute plan one phase at a time
    test/SKILL.md              # Utility: run automated checks in parallel
    review/SKILL.md            # Utility: code review at low/medium/high depth
    commit/SKILL.md            # Utility: stage and commit with task reference
    status/SKILL.md            # Utility: show task progress
```

## Architecture

**Plugin structure**: The `dev/` directory is a Claude Code plugin. `plugin.json` defines the plugin metadata. Skills are namespaced as `/dev:<skill-name>` (e.g., `/dev:setup`, `/dev:research`).

**Phases vs Utilities**: The four phases (setup → research → plan → implement) form a sequential loop with human checkpoints between each. Utility skills (test, review, commit, status) can run at any time.

**Agents**: The `agents/` directory contains subagent definitions that skills invoke via the Agent tool. Research and review skills at medium/high intensity delegate to specialized agents for parallel analysis. The `test-runner` agent is a lightweight (haiku model) agent that executes a single check command — the `/dev:test` skill launches multiple test-runner instances in parallel. The implement skill can also delegate exploration or pattern analysis to agents when it would save context or improve results.

**Skill file format**: Each `SKILL.md` has YAML frontmatter with `description`, `disable-model-invocation: true`, and `allowed-tools` (granular per-skill tool permissions). The body is markdown instructions that Claude follows when the skill is invoked.

**Agent file format**: Each agent `.md` file has YAML frontmatter with `name`, `description`, `tools`, `model`, and `maxTurns`. The body is a system prompt defining the agent's behavior. Agent tool access is scoped narrowly (e.g., test-runner only gets `Bash`). Skills restrict which agents they can spawn via `Agent(agent-name, ...)` in `allowed-tools`.

**Task artifacts**: When the plugin is used in a work repo, it creates `task_<task-id>/` at the repo root containing `state.json`, `context.md`, `research.md`, `plan.md`, and optionally `review.md`. One task per branch; task directory is removed before merging to main. The `state.json` includes a `testCommands` field mapping check names to commands (e.g., `{"lint": "eslint . --fix --quiet"}`).

## Conventions

- Skills have no `name` field in frontmatter — namespacing comes from the directory name under `skills/` combined with the plugin name.
- `allowed-tools` should be scoped as narrowly as possible per skill (principle of least privilege). Bash commands use glob patterns (e.g., `Bash(git diff *)`). Agent access is scoped to named agents (e.g., `Agent(test-runner)`).
- `disable-model-invocation: true` is set on all skills to prevent automatic triggering — skills must be explicitly invoked.
- Commits use Conventional Commits format (see `dev/skills/commit/SKILL.md`).
- Test commands combine fix + quiet flags where possible (e.g., `eslint --fix --quiet`) so a single invocation auto-fixes and reports only remaining issues.
