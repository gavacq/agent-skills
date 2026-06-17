# agent-skills

Personal experimental Claude Code plugins.

This repo contains custom plugins — bundling skills, agents, hooks, and MCP servers — that I use to streamline development with coding agents. These are opinionated, iterative, and built for my own workflows — not intended as general-purpose tooling.

## Plugins

| Directory | Plugin | Purpose |
|-----------|--------|---------|
| `dev/`    | `dev`  | Development workflow plugin — phased loops for building features and fixing bugs |
| `pr/`     | `pr`   | PR workflow — split large branches into reviewable stacks |

## Usage

Load a plugin locally for testing:

```bash
claude --plugin-dir ./dev
claude --plugin-dir ./pr
```

### Symlinks (Cursor + Claude Code)

Canonical skill files live in this repo (`~/src/personal/agent-skills`, also linkable as `~/src/agent-skills`). Point both tools at the same directory — do not edit copies under `~/.cursor/skills/` unless they are symlinks:

```bash
REPO="$HOME/src/personal/agent-skills"
SKILL_SRC="$REPO/pr/skills/split-large-prs"

ln -sfn "$SKILL_SRC" "$HOME/.cursor/skills/split-large-prs"
ln -sfn "$SKILL_SRC" "$HOME/.claude/skills/split-large-prs"
```

Optional convenience symlink if you prefer `~/src/agent-skills`:

```bash
ln -sfn "$HOME/src/personal/agent-skills" "$HOME/src/agent-skills"
```

After symlinking, use `/split-large-prs` in Cursor or Claude Code. Restart the session if the skill does not appear.
