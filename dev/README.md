# dev plugin

Claude Code plugin for phased development workflows — building features and fixing bugs with configurable depth.

## The Loop

```
Setup → Research → Plan → Implement
```

Each phase is a checkpoint. The AI pauses after each phase for human review before proceeding.

## Installation

Load the plugin from your local clone of agent-skills (not committed to the work repo):

```bash
claude --plugin-dir /path/to/agent-skills/dev
```

## Tasks

Each branch has one task. The task directory lives at the work repo root:

```
task_<task-id>/
  state.json       # phase tracking, config, timestamps
  context.md       # original task description and Q&A from setup
  research.md      # findings from research phase
  plan.md          # phased implementation plan with checkboxes
```

Task files are committed to the branch for tracking and shareability during development. **Remove the task directory before merging to main.**

## Phases

### `/dev:setup [task-id-or-context]`
- Interactive Q&A to gather context: environment, domain, scope, relevant MCPs/tools
- Generates a task ID from the conversation (or uses one you provide)
- Creates the task directory with `state.json` and `context.md` at the repo root
- Accepts arbitrary input (markdown, Linear tickets, PRs, etc.)

### `/dev:research <task-id> [low|medium|high]`
- **Low**: Quick scan of relevant files, surface-level understanding
- **Medium**: Sub-agents for codebase analysis, pattern finding, dependency mapping
- **High**: Deep multi-agent research — schema analysis, API surface mapping, cross-repo exploration
- Output: `research.md` in the task directory

### `/dev:plan <task-id>`
- Generates a phased markdown plan with checkboxes
- Each phase maps to an implementable unit of work
- Saved as `plan.md` in the task directory
- Editable — phases can be added, removed, or reordered before execution

### `/dev:implement <task-id>`
- Executes the next incomplete phase of the plan
- Marks completed steps with checkboxes in `plan.md`
- Stops at the end of each phase for review
- Automated checks run after each phase: lint, test, typecheck

## Utility Skills

### `/dev:review <task-id> [low|medium|high]`
- **Low**: Quick diff scan, basic correctness check
- **Medium**: Code review with suggestions, edge case analysis
- **High**: Comprehensive review — architecture alignment, performance, security
- Output: `review.md` in the task directory

### `/dev:commit <task-id>`
- Summarizes changes and drafts a commit message referencing the task ID
- Stages code changes and task directory files
- Commits after user approval

### `/dev:status [task-id]`
- With a task ID: show phase progress and config
- Without: list all task directories at repo root

## Task Tiers

| Tier | Use Case | Default Research | Default Review |
|------|----------|-----------------|----------------|
| Small | Bug fixes, tweaks | Low | Low |
| Medium | New features, refactors | Medium | Medium |

## Conversational

The loop is designed to be conversational. At any point you can:
- Ask clarifying questions about the current phase
- Adjust research/review intensity
- Skip or reorder phases
- Run utility skills (`/dev:review`, `/dev:commit`, `/dev:status`) at any time
