# Delegation Playbook (read before spawning any sub-agent)

Loaded on demand by the Working Rules. Every sub-agent starts cold and **every tool call
re-sends its whole context** (system prompt, tool schemas, CLAUDE.md, everything read so far),
so cost ≈ turns × context size. Cut both.

## Which agent

| Situation | Agent |
|---|---|
| Broad search, only the conclusion needed | `Explore` (skips CLAUDE.md, cheap) |
| Bounded implementation/test/fix in known files | Project worker agent; if none, create one (below) or pass `model: sonnet` |
| Several independent bounded tasks | Parallel agents in ONE message, each owning **disjoint files** |
| Phase 3 final review | Project finalizer agent; if none, create one (below) |

Avoid `general-purpose` whenever a restricted agent exists: it inherits every MCP/plugin tool schema.

## Brief template

```
TASK:        <one sentence, the outcome>
FILES:       <path>:<lines> — <why>   (every file it may touch)
FACTS:       <what I already know: names, fields, root cause, decisions> — don't re-derive
DO:          <numbered concrete steps>
DON'T:       read docs/ or unnamed files; refactor outside scope; <task traps>
TEST:        <exact targeted command(s), not the full suite>
DONE WHEN:   <verifiable criteria — the plan step's "verify" check>
BUDGET:      <max tool calls: simple fix 5–10, 1–2 file feature 15–25; bigger → split>
OUTPUT FILE: <scratch path for bulky results; return only path + 1-line summary>
REPORT:      ≤15 lines: DONE/BLOCKED · changed file:line · tests pass/fail · open issues
```

- Pre-digest context: paste the 5–20 relevant lines instead of "read X to understand".
- Workers with `omitClaudeMd` don't see CLAUDE.md, so put any project fact they need in FACTS.
- One task per agent. Two tasks on the same file → sequential, or separate worktrees.
- BLOCKED or wrong → prefer a small inline fix; else continue the **same** agent (SendMessage)
  with a sharper brief rather than a cold re-spawn.
- Verify with `git diff --stat` + targeted reads, not a full re-read.

Finalizer brief: `TASK: Phase 3 final review · PLAN: <path> · BASE: <commit> · TESTS: <cmds>`
(re-review: `TASK: RE-REVIEW after FAIL` + previous findings; check only those + the fixes).

## Creating agents in `.claude/agents/` (when a project has none)

**Worker** frontmatter:
```
tools: Read, Edit, Write, Grep, Glob, Bash
disallowedTools: mcp__*
model: sonnet
effort: medium
maxTurns: 40
omitClaudeMd: true
```
Its prompt (keep short): project basics it needs (run/test commands, key conventions, safety
limits); read line ranges not whole files; `grep -n` before reading; pipe long output through
`tail`; run only named tests; batch independent calls; honor the budget; stop at done-criteria;
never use production/live flags; report in the 4-point format above.

**Finalizer**: same, but `tools: Read, Grep, Glob, Bash`, `model: opus`, `effort: high`,
`maxTurns: 60`, no Edit/Write. Prompt: read the plan + `git diff <base>`; check done-criteria →
bugs/edge cases → project safety rules → real simplification payoff (no style nitpicks); run the
plan's tests; report ≤40 lines: PASS / PASS WITH FIXES / FAIL, findings ranked
`file:line — problem — fix`.
