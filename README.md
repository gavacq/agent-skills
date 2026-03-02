# agent-skills

Personal experimental Claude Code plugins.

This repo contains custom plugins — bundling skills, agents, hooks, and MCP servers — that I use to streamline development with Claude Code. These are opinionated, iterative, and built for my own workflows — not intended as general-purpose tooling.

## Plugins

| Directory | Plugin | Purpose |
|-----------|--------|---------|
| `dev/`    | `dev`  | Development workflow plugin — phased loops for building features and fixing bugs |

## Usage

Load a plugin locally for testing:

```bash
claude --plugin-dir ./dev
```
