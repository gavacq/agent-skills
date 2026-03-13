---
description: Stage all changes and commit with a well-structured conventional commit message. Fire and forget — no confirmation prompts.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git *)
---

# Commit

You are committing changes for a development task. This is a fire-and-forget operation — do NOT prompt the user for confirmation at any point. Stage everything, draft the message, and commit immediately.

## 1. Gather context from conversation

Check the conversation for task context (from `/dev:setup` or `/dev:task load`). If available, note:
- Task ID (for the `Task:` footer)
- Commit scopes (for scope selection)
- Plan (for understanding intent)

This context is optional — the commit works without it.

## 2. Check what changed

Run `git status` and `git diff` to understand the changes.

## 3. Stage all changes

Run `git add .` from the repo root. Stage everything — do not cherry-pick files.

## 4. Draft and commit

Use **Conventional Commits** format:

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

- If a task ID is available in the conversation, include `Task: <task-id>` as a footer.
- Keep messages concise — prefer a clear one-line description over a verbose body.
- If commit scopes are available from conversation context, prefer scopes from that list. Use scope when the change targets a specific module, component, or subsystem and the description alone wouldn't make that clear.

Commit immediately — do NOT ask the user to review or approve the message. Use `-m` flags directly. For multi-line messages, use multiple `-m` flags (one per paragraph).

## 5. Summary

Tell the user:
- The commit hash and message
- Suggest next steps: push, create PR, or start a new task
