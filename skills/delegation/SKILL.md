---
name: delegation
description: Playbook for spawning sub-agents with the Agent tool — which agent and model, brief template, budgets, report format, agent definitions. Load before the first sub-agent spawn in a session, for any task type (research, documents, code).
---

# Delegation Playbook

Every sub-agent starts cold and **every tool call re-sends its whole context** (system prompt,
tool schemas, CLAUDE.md, everything read so far), so cost ≈ turns × context size. Cut both. A sub-agent also doesn't read the parent's prompt
cache and gets a 5-minute cache lifetime, so it only pays off for bulky or parallel work.

## Which agent

| Situation | Agent | Model / effort |
|---|---|---|
| Broad codebase search, only the conclusion needed | `Explore` | `sonnet` |
| Web or documentation research | project scout agent; if none, create one (below) | `sonnet` / `medium` |
| Independent read-only tasks (search, review, competing debug hypotheses, research subtopics) | parallel agents in ONE message | `sonnet` / `medium`–`high` |
| Bounded implementation/test/fix in known files | project worker agent; if none, create one (below) | `sonnet` / `high` |
| Several code-writing tasks | one worker at a time, even on disjoint files; never parallel writers | `sonnet` / `high` |
| Sub-task needs this conversation's context | fork (reads the parent's cache) | parent's model |
| Final review (code, or a document others will act on) | project finalizer agent; if none, create one (below) | `opus` / `high` |

- Pass `model` on every Agent call. Built-in `Explore` and `general-purpose` otherwise inherit
  the session model (Opus). `Explore` skips CLAUDE.md, so put project facts in FACTS.
- The Agent call has no effort parameter: effort comes from agent frontmatter, else it inherits
  the session's. When effort matters, use a defined agent.
- Never `effort: max` on any agent: highest token burn, spawns nested sub-agents, drifts out of scope;
  Sonnet 5.5 at max costs more per task than Opus 5.5, and in one reported test burned 128K tokens
  with no answer.
- A worker that waits on long tests: consider `experimental: {cacheTtl: 1h}` in its frontmatter.
- Avoid `general-purpose` whenever a restricted agent exists: it inherits every MCP/plugin tool schema.

## Brief template

```
TASK:        <one sentence, the outcome>
FILES:       <path>:<lines> or <URL> — <why>   (every file it may read or touch)
FACTS:       <what I already know: names, fields, root cause, decisions> — don't re-derive
DO:          <numbered concrete steps>
DON'T:       read unnamed files; edit anything (read-only helpers); work outside scope; <task traps>
SCOPE:       exhaustive (every item, no sampling) | sample OK — if you must sample, say so before starting
CHECK:       <exact targeted command(s), not the full suite> | research: file:line or URL for every claim
DONE WHEN:   <verifiable criteria — the plan step's "verify" check>
BUDGET:      <max tool calls: lookup 3–5 · simple fix 5–10 · research subtopic 10–20 · 1–2 file feature 15–25; bigger → split>
OUTPUT FILE: <scratch path for bulky results; return only path + 1-line summary; past ~15 tool calls, append progress to it as you go>
REPORT:      ≤15 lines: DONE/BLOCKED · changed file:line or findings with sources · files checked · check pass/fail · open issues
```

- Pre-digest context: paste the 5–20 relevant lines instead of "read X to understand".
- Research helpers mark each finding verified (source opened) or inferred, and report every
  "not found" with where they looked (paths, queries, URLs) so it can be checked.
- One task per agent. Two tasks on the same file → sequential.
- BLOCKED or wrong → prefer a small inline fix; else continue the **same** agent (SendMessage)
  with a sharper brief rather than a cold re-spawn.
- Verify with `git diff --stat` + targeted reads, not a full re-read, and re-run the CHECK
  yourself: a helper's "tests pass" or "source says X" is a claim, not evidence.

Finalizer brief: `TASK: final review · PLAN: <path> (or REQUEST: <my request> + FILE: <path>) · BASE: <commit> · CHECKS: <cmds>`
(re-review: `TASK: RE-REVIEW after FAIL` + previous findings; check only those + the fixes).

## Agent definitions (when a project has none)

Put them in the project's `.claude/agents/` (committed files also load in cloud sessions) or
`~/.claude/agents/` (this machine only). Adding them to a repo is a change: mention it in the report.

**Worker**:
```
tools: Read, Edit, Write, Grep, Glob, Bash
disallowedTools: mcp__*
model: sonnet
effort: high
maxTurns: 40
omitClaudeMd: true
```
Prompt (keep short): project basics it needs (run/test commands, key conventions, safety
limits); read line ranges not whole files; `grep -n` before reading; pipe long output through
`tail`; run only named tests; batch independent calls; honor the budget; stop at done-criteria;
never use production/live flags; never spawn sub-agents; report in the format above.

**Scout** (read-only research): `tools: Read, Grep, Glob, WebFetch, WebSearch`, `disallowedTools: mcp__*`, `model: sonnet`,
`effort: medium`, `maxTurns: 25`, `omitClaudeMd: true`. Prompt: answer only the brief's question;
cite file:line or URL for every claim; mark each claim verified or inferred; never edit; never
spawn sub-agents; report in the format above.

**Finalizer**: `tools: Read, Grep, Glob, Bash`, `disallowedTools: mcp__*`, `model: opus`, `effort: high`, `maxTurns: 60`,
no Edit/Write. Prompt: read the plan + `git diff <base>` (or the request + the document); check
done-criteria → bugs/edge cases or factual errors → project safety rules → real simplification
payoff (no style nitpicks); run the checks; report ≤40 lines: PASS / PASS WITH FIXES / FAIL,
findings ranked `file:line — problem — fix`.
