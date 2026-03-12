---
description: Review code changes or a planning doc. Operates on unstaged changes (default), a commit range, or a plan file. Intensity is configurable (low, medium, high).
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Edit, Write, Skill, Agent(correctness-reviewer, security-reviewer, architecture-reviewer), Bash(git diff *), Bash(git log *)
---

# Review

You are reviewing code for a development task. `$ARGUMENTS` contains positional arguments and flags:

```
/dev:review <task-id> [low|medium|high] [--commits <base>..<head>] [--plan]
```

- No flag: review current unstaged changes (`git diff`)
- `--commits <base>..<head>`: review the diff between two commits (`git diff <base>..<head>`)
- `--plan`: review the planning doc (`task_<task-id>/plan.md`) instead of code

## 1. Load task state (optional)

If `$ARGUMENTS` contains a task ID (e.g. `my-task-123`), read `task_<task-id>/state.json` and `task_<task-id>/plan.md`. If those files don't exist, warn the user and continue without task context.

If no task ID is recognizable in `$ARGUMENTS` (e.g. the user passed "current PR", a commit range, or just a flag), proceed without task context. Do NOT stop.

## 2. Determine what to review

| Mode | Source |
|---|---|
| Unstaged (default) | `git diff` |
| Commit range | `git diff <base>..<head>` from `--commits` flag |
| "current PR" or "PR" | `git diff main..HEAD` (or `master..HEAD` if main doesn't exist) |
| Plan | Contents of `task_<task-id>/plan.md` |

For code modes (unstaged / commit range / PR), collect the full diff and identify every changed file.

For plan mode, skip to §5 (plan review).

## 3. Run automated checks

Use the **Skill tool** to invoke `/dev:test`, passing the task-id if one is available, or no arguments if not:

- With task: `/dev:test <task-id>`
- Without task: `/dev:test`

The test skill will resolve commands automatically. Wait for it to complete and capture the results.

**Do NOT run test commands via Bash directly.** Always use the Skill tool here — the test skill handles command discovery and parallel execution.

## 4. Code review

### Determine intensity

Use the intensity from `$ARGUMENTS` if provided, otherwise fall back to `reviewLevel` in state.json. If neither is available, default to `medium`.

### Review areas (in priority order)

Evaluate every area below that is relevant to the diff. Earlier areas take priority when reporting findings.

1. **Correctness** — Does the code do what it claims? Are there logic errors, off-by-one mistakes, race conditions, missing error handling, or unhandled edge cases?
2. **Performance** — Are there unnecessary allocations, O(n²) loops, missing indexes, repeated expensive operations, or opportunities to batch/cache?
3. **UX / UI / Accessibility** — (Only when the diff touches user-facing code.) Are interactions intuitive? Is there proper aria labelling, keyboard navigation, colour contrast, loading/error states?
4. **Security** — Injection risks (SQL, XSS, command), auth/authz gaps, secret leakage, insecure defaults, missing input validation at trust boundaries?
5. **Refactoring & design** — Dead code, duplicated logic that warrants extraction, naming clarity, adherence to codebase conventions found during research, SOLID / DRY opportunities that aren't over-engineering?

### Intensity levels

#### Low
- Quick scan of the diff — flag anything in areas 1-2 only
- Confirm changes align with the plan

#### Medium
- Review each changed file against all five areas
- Summarize findings with specific file:line references

#### High
- Launch three review agents in parallel:
  - `correctness-reviewer`: Correctness, logic errors, edge cases, performance (areas 1-2)
  - `security-reviewer`: Security vulnerabilities, UX/accessibility concerns (areas 3-4)
  - `architecture-reviewer`: Architectural consistency, refactoring opportunities, design patterns (area 5)
- Provide each agent with:
  - The full diff being reviewed
  - The plan from `task_<task-id>/plan.md`
  - Research findings from `task_<task-id>/research.md` (if available, for pattern context)
- Cross-reference findings across all three agents
- Deduplicate overlapping findings
- Produce a comprehensive review document organized by severity

## 5. Plan review (--plan)

When `--plan` is passed, review `task_<task-id>/plan.md` for:

- **Completeness** — Does the plan cover all acceptance criteria from `context.md`?
- **Correctness** — Are the proposed changes technically sound?
- **Risk** — Are there risky steps that need mitigation or fallback?
- **Ordering** — Is the phase sequence logical? Are dependencies respected?
- **Scope** — Is the plan over-engineered or missing necessary work?

## 6. Write review findings

If a task ID was provided and the task directory exists, create or update `task_<task-id>/review.md` with the findings below. Otherwise, skip writing to disk and present findings directly (§7).

When writing, include:

- Review mode (unstaged / commit range / plan) and intensity
- Summary (pass / fail / needs-changes)
- Issues found (with file:line references for code, section references for plan) and severity
- Suggestions for improvement
- Automated check results from `/dev:test` (code reviews only)

## 7. Present findings

Show the user the review results. If issues were found:
- List them clearly with severity
- Suggest fixes
- Ask the user how to proceed (fix and re-review, accept as-is, etc.)

If the review passes, suggest `/dev:commit <task-id>`.
