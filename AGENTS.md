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
  skills/                      # Each subdirectory is a skill
    setup/SKILL.md             # Phase: initialize task, gather context
    research/SKILL.md          # Phase: explore codebase at low/medium/high depth
    plan/SKILL.md              # Phase: generate phased implementation plan
    implement/SKILL.md         # Phase: execute plan one phase at a time
    review/SKILL.md            # Utility: code review at low/medium/high depth
    commit/SKILL.md            # Utility: stage and commit with task reference
    status/SKILL.md            # Utility: show task progress
```

## Architecture

**Plugin structure**: The `dev/` directory is a Claude Code plugin. `plugin.json` defines the plugin metadata. Skills are namespaced as `/dev:<skill-name>` (e.g., `/dev:setup`, `/dev:research`).

**Phases vs Utilities**: The four phases (setup → research → plan → implement) form a sequential loop with human checkpoints between each. Utility skills (review, commit, status) can run at any time.

**Skill file format**: Each `SKILL.md` has YAML frontmatter with `description`, `disable-model-invocation: true`, and `allowed-tools` (granular per-skill tool permissions). The body is markdown instructions that Claude follows when the skill is invoked.

**Task artifacts**: When the plugin is used in a work repo, it creates `task_<task-id>/` at the repo root containing `state.json`, `context.md`, `research.md`, `plan.md`, and optionally `review.md`. One task per branch; task directory is removed before merging to main.

## Conventions

- Skills have no `name` field in frontmatter — namespacing comes from the directory name under `skills/` combined with the plugin name.
- `allowed-tools` should be scoped as narrowly as possible per skill (principle of least privilege). Bash commands use glob patterns (e.g., `Bash(git diff *)`).
- `disable-model-invocation: true` is set on all skills to prevent automatic triggering — skills must be explicitly invoked.
- Commits use `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>` footer.
