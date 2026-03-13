---
description: Autonomous dev agent — accepts a task and handles the full workflow
disable-model-invocation: true
allowed-tools: Agent(dev-agent)
---

# Auto

You are dispatching an autonomous dev agent to handle a task end-to-end.

## 1. Determine the task

Check `$ARGUMENTS` for a task description. If empty, check the conversation for existing task context (from `/dev:setup`, `/dev:task load`, or user messages). If no task can be determined, ask the user what they want done.

## 2. Gather conversation context

Subagents do not inherit the parent conversation. Before launching the agent, collect any relevant context already in the conversation:

- **Task setup data** — task ID, tier, scope, commit scopes, test commands, constraints (from `/dev:setup` or `/dev:task load`)
- **Research findings** — codebase exploration results, patterns, prior work (from `/dev:research`)
- **Plan** — implementation plan with phases and steps (from `/dev:plan`)
- **Review results** — any prior review output (from `/dev:review`)

Only include sections that actually exist in the conversation. Do not fabricate context.

## 3. Launch the dev-agent

Spawn the `dev-agent` with a structured prompt:

> Agent(dev-agent):
> ## Task
> <task description from $ARGUMENTS or conversation>
>
> ## Existing Context
> <any context gathered in step 2, or "None — starting fresh">

## 4. Report results

When the agent completes, relay its results to the user:
- What was accomplished
- Any issues encountered or decisions made
- The commit hash and message (if changes were committed)
- Any remaining work or suggested follow-ups
