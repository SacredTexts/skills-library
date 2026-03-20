# Skills Library — Agent Configuration

## Project Overview

Skills library for agent skill distribution and management. Pure markdown-driven system — no build tools required.

## Agent References

- `agents/coder.md` — Primary coding agent
- `agents/tester.md` — Testing agent
- `agents/stuck.md` — Unstuck helper agent
- `agents/TASK-MANAGEMENT.md` — Task tracking protocol
- `agents/SDK-INTEGRATION.md` — SDK integration patterns

## Commands

Available slash commands in `.claude/commands/`:
- `/start` — Dev environment startup
- `/pr` — Pull request workflow
- `/commit-message` — Commit message generation
- `/fix-issue` — Issue fixing workflow
- `/branch-new` — Create new branch
- `/goldy` — Gold standard planning
- `/goldy-loop` — Continuous execution
- `/status` — Status reporting
- `/ultra-think` — Extended reasoning

## Skills

Installed skills in `.claude/skills/` — see individual skill directories for documentation.

## CLI Tools

- **GitHub CLI** (`gh`) — PR management, issue tracking, repo operations
- **Vercel CLI** (`vercel`) — Deployment, env management, project linking
- **Wrangler CLI** (`wrangler`) — Cloudflare Workers, D1, KV, R2, Pages

## Conventions

- Use Gold Standard plan structure for all plans
- Follow checklist rule: mark `[x]` only when validated
- Session-start audit before continuing any plan
