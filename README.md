# Claude Code working rules

| File | What it is | Where to install |
|---|---|---|
| `docs/working-rules.md` | Always-on rules (who works, tracks, delegation nevers, guidelines) | claude.ai → Settings → Profile → personal preferences (reaches cloud sessions); also `~/.claude/CLAUDE.md` for local terminal sessions |
| `skills/delegation/SKILL.md` | On-demand playbook, loaded only before spawning sub-agents | Upload as a skill to your claude.ai account (syncs to cloud, Cowork and signed-in terminal); or copy to `~/.claude/skills/delegation/` (this machine only) |
| `.claude/agents/` | Sonnet `scout` and `worker` definitions (from the delegation skill) | Load automatically in this repo; copy to another project's `.claude/agents/` (edit the worker's project-basics line) or to `~/.claude/agents/` (this machine only) |
| `docs/research/` | Evidence behind the rules (Opus 5.5 vs Sonnet 5.5, 2026-09-29) | Reference only |

Revisit when Claude Haiku 5.5 ships (announced on 2026-09-28 for "the coming weeks"): consider it for lookup-only helpers.

One-time setup that supports Session hygiene:
- In each project's `CLAUDE.md`, add: `Compact instructions: preserve the plan path and current step.` (a plan file in `docs/plans/` is not re-injected after compaction).
- Optional safety net: run `/autocompact 400k` so auto-compaction triggers well before the ~967K default.
- Optional: set `CLAUDE_CODE_SUBAGENT_MODEL=sonnet` (cloud: environment menu in the session title bar → Edit → environment variables; local: `env` in settings.json) so a spawn that omits its model still runs on Sonnet. A per-call model or an agent's `model` field wins, so the Opus finalizer is unaffected.
- Don't set `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1`: it would also force the Opus finalizer onto Sonnet.
