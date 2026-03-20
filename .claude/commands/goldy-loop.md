---
allowed-tools: Bash, Read, Write, Edit, MultiEdit, Glob
disable-model-invocation: true
description: Run GOLDY phase loop with checkpoints and auto-resume chain
argument-hint: --plan <plan-file> [options]
---

# GOLDY LOOP

Run GOLDY loop engine:

```bash
python3 /Users/forest/.agents/skills/goldy/scripts/goldy_loop.py $ARGUMENTS
```

Behavior:
- Manual-only command; do not auto-invoke from `/goldy`.
- Guardrail-compliant stop on low context.
- Creates/reuses a session git worktree by default.
- Requires a user-authored plan outside `temp-plans/` (unless `--allow-temp-plan` is set).
- Worktree identity is derived from the plan file, so different plan files use different worktrees.
- Enforces plan drift detection; use `--require-resync` to sync source plan into mapped worktree plan copy.
- Runs preflight ambiguity checks and asks clarifying questions before executing the loop.
- Interactive sessions offer `Start` and `Chat` after preflight; non-interactive runs should pass `--mode` explicitly and otherwise default to `start`.
- Runs up to 10 loop iterations by default (`--max-iterations` to override).
- Runs 5 deep audits (lint, typecheck, tests, integration, robustness) before final completion.
- Prints a completion report including total compactions and total minutes.
- Writes checkpoints, append-only history, lock metadata, diagnostics (optional), and deterministic handoff artifacts.
- Supports resume via session id.
- Supports breaker operator controls (`--breaker-status`, `--breaker-reset`, `--breaker-auto-reset`).
- Supports `--diagnostics` for per-session diagnostic bundles in `.goldy/diagnostics`.
- Optional phase-level commits with `--commit-phase`.
- `--commands` prints full command list + usage examples.
- Running without `--plan` prints the same command reference.
