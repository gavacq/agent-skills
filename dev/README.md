# Dev Skills

Development workflow skills for Claude Code — a phased loop for building features and fixing bugs with configurable depth.

## The Loop

```
Setup → Research → Plan → Implement → Review → Commit
```

Each phase is a checkpoint. The AI pauses after each phase for human review before proceeding.

## Phases

### 1. Setup (`/dev setup`)
- Interactive Q&A to gather context: environment, domain, scope, relevant MCPs/tools
- Persists task context to a planning document and state JSON
- Accepts arbitrary input (markdown, Linear tickets, PRs, etc.)

### 2. Research (`/dev research [low|medium|high]`)
- **Low**: Quick scan of relevant files, surface-level understanding
- **Medium**: Sub-agents for codebase analysis, pattern finding, dependency mapping
- **High**: Deep multi-agent research — schema analysis, API surface mapping, cross-repo exploration
- Output: research summary appended to the planning document

### 3. Plan (`/dev plan`)
- Generates a phased markdown plan with checkboxes
- Each phase maps to an implementable unit of work
- Plan is saved to the repo branch as a `.md` file
- Editable — phases can be added, removed, or reordered before execution

### 4. Implement (`/dev implement`)
- Executes the current phase of the plan
- Marks completed steps with checkboxes
- Stops at the end of each phase for review
- Automated checks run after each phase: lint, test, typecheck

### 5. Review (`/dev review [low|medium|high]`)
- **Low**: Quick diff scan, basic correctness check
- **Medium**: Code review with suggestions, edge case analysis
- **High**: Comprehensive review — architecture alignment, performance, security
- Can be run between any phases on demand

### 6. Commit (`/dev commit`)
- Summarizes changes
- Creates a well-structured commit message
- Stages and commits relevant files

## Task Tiers

| Tier | Use Case | Default Research | Default Review |
|------|----------|-----------------|----------------|
| Small (`/dev task small`) | Bug fixes, tweaks | Low | Low |
| Medium (`/dev task medium`) | New features, refactors | Medium | Medium |

## State Tracking

State is tracked in JSON (`.dev-state.json`) at the repo root:
- Current phase
- Research/review levels
- Plan file path
- Completed phases
- Active MCPs/tools

Plans are tracked in markdown with checkboxes for incremental progress.

## Conversational

The loop is designed to be conversational. At any point you can:
- Ask clarifying questions about the current phase
- Adjust research/review intensity
- Skip or reorder phases
- Inspect state with `/dev status`
