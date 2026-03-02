---
description: Stage and commit changes for the current task with a well-structured commit message.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git *)
---

# Commit

You are committing changes for a development task. The task ID is in `$ARGUMENTS`.

## 1. Load task state

Read `task_<task-id>/state.json`, `task_<task-id>/context.md`, and `task_<task-id>/plan.md`.

## 2. Check readiness

- Run `git status` (to see modified and untracked files) and `git diff` to see what's changed
- Do NOT run automated checks — use `/dev:test` separately if needed

## 3. Stage changes

Stage the relevant files. Include:
- All modified files related to the task
- All untracked files created as part of the task (new files, new directories)
- The task directory files (`task_<task-id>/`)
- Do NOT stage files that contain secrets (.env, credentials, etc.)

Use specific file paths rather than `git add -A`.

## 4. Draft commit message

Use **Conventional Commits** format. Every commit MUST follow this structure:

```
<type>[(scope)]: <description>

[optional body]

[optional footer(s)]
```

### Specification

1. The commit MUST be prefixed with a type (`feat`, `fix`, etc.), an optional scope in parentheses, an optional `!` (for breaking changes), followed by a colon and space.
2. `feat` — adds a new feature.
3. `fix` — patches a bug.
4. Scope is optional — a noun in parentheses describing the affected section (e.g., `feat(auth):`, `fix(parser):`). Use scope when it adds clarity to a short message; omit when the description is already unambiguous.
5. Description MUST immediately follow the colon+space. Keep it under 72 chars, imperative mood, lowercase start, no trailing period.
6. Body is optional — one blank line after description. Explain the "why" if not obvious.
7. Footers follow one blank line after the body, using `token: value` format (hyphens replace spaces in tokens, e.g., `Reviewed-by:`).
8. Breaking changes: use `!` after type/scope (e.g., `feat!:`) and/or a `BREAKING CHANGE: <description>` footer.
9. Types beyond `feat` and `fix` are permitted: `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.

### Rules for this skill

- Always include `Task: <task-id>` as a footer.
- Keep messages concise — prefer a clear one-line description over a verbose body.
- If `commitScopes` in state.json is non-empty, prefer scopes from that list. Use scope when the change targets a specific module, component, or subsystem and the description alone wouldn't make that clear.
- If the commit only updates task artifact files (`task_<task-id>/`), use `_task` as the scope (e.g., `chore(_task): update plan after research`).

Present the commit message to the user for approval before committing.

## 5. Commit

After user approval, create the commit. Do NOT use command substitution (`$(...)`) or heredocs in the `git commit` command — pass the message directly with `-m` flags. For multi-line messages, use multiple `-m` flags (one per paragraph).

## 6. Update state

Update `task_<task-id>/state.json`:
- Update `updated` timestamp
- Do NOT modify `phase` or `phases` — commit is a utility, not a phase

## 7. Summary

Tell the user:
- The commit hash and message
- Suggest next steps: push, create PR, or start a new task
