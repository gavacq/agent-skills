---
description: Research the codebase and gather information. Intensity is configurable (low, medium, high).
allowed-tools: Read, Write, Edit, Glob, Grep, Agent(codebase-explorer, pattern-analyzer, prior-work-searcher), mcp__context7__*
---

# Research

You are conducting research for a development task. `$ARGUMENTS` contains an optional intensity level and the research topic.

Examples:
- `/dev:research how does the auth middleware work`
- `/dev:research high where are API routes defined`
- `/dev:research low what tests exist for the parser`

## 1. Parse arguments

Parse `$ARGUMENTS` to extract:
- **Intensity**: one of `low`, `medium`, `high` (if the first word matches)
- **Topic**: all remaining text is the research topic or question

If no intensity is specified, default to `medium`.

If `$ARGUMENTS` is empty, check the conversation for task context (e.g., from a previous `/dev:task load` or `/dev:setup`). Use the task description as the research topic. If no topic can be determined, ask the user what to research.

## 2. Conduct research

### Low
- Read the files directly relevant to the topic
- Summarize what exists, how it works, and what needs to change
- Quick and focused — stay in the main conversation

### Medium
- Launch a `codebase-explorer` agent with the research topic
- The agent will map structure, trace dependencies, and identify relevant files, tests, types, and utilities
- Summarize the agent's findings with file paths and line references

### High
- Launch three agents in parallel:
  - `codebase-explorer`: Map the codebase structure and dependencies relevant to the topic
  - `pattern-analyzer`: Analyze coding patterns, conventions, and style norms in the relevant areas
  - `prior-work-searcher`: Search for related prior work, documentation, and existing implementations
- Cross-reference findings across all three agents
- Produce a comprehensive research document with architecture notes, dependency maps, and implementation recommendations

## 3. Present research findings

Include:
- Summary of findings
- Key files and their roles (with paths and line numbers)
- Patterns and conventions to follow
- Dependencies and potential impacts
- Open questions or risks

## 4. Suggest next steps

Tell the user what was found. Suggest:
- `/dev:plan` to generate an implementation plan from the research
- `/dev:task save research` to persist the findings (if a task is active in the conversation)
