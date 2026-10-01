# Make Sonnet 5.5 helpers actually get used

Base commit: 4b50701

## Goal
Sessions that follow these rules spawn Sonnet 5.5 sub-agents for work whose scope fits Sonnet
(per docs/research/2026-09-29-opus-vs-sonnet-subagents.md:62-70), and keep everything else on Opus.

## Why Sonnet is never used today (verified 2026-10-01)
- The cloud harness tells the model: "Do not spawn agents unless the user asks ... Only use this tool
  when the user explicitly says to use a subagent, or names one of the available agent types."
- working-rules.md:24 "Default is no helper. Spawn one only to ..." is permissive, so both point to inline.
- No agent definitions exist (no .claude/agents/, no ~/.claude/agents/); built-ins inherit Opus.
- delegation skill says "if none, create one", but a new agents/ directory loads only after a
  restart (code.claude.com/docs/en/sub-agents), so that fallback fails mid-session.

## Assumptions / decisions
- Agent definitions go in this repo's .claude/agents/ (load in every session of this repo, cloud
  included); README tells how to copy them to other projects. Other projects without definitions use
  built-in Explore/general-purpose with model sonnet. (Account-wide plugin: not done; possible later.)
- Alias `sonnet` = Sonnet 5.5 on the Anthropic API (code.claude.com/docs/en/model-config).
- "More than ~5 tool calls" is the size cue for spawning a scout; it is my choice, not from research.
- The standing request must be explicit and name agent types, to meet the harness condition.

## Steps
1. working-rules.md:24: replace the "Default is no helper" bullet with a standing request, a
   "Sonnet fits" list (research table) and a "stays on Opus" list → verify: diff touches only that
   bullet; track table and Change-track rules still agree with it.
2. Add .claude/agents/scout.md and worker.md from skills/delegation/SKILL.md:67-84, descriptions say
   "Use proactively" → verify: every frontmatter key is in the documented field list; model sonnet.
3. skills/delegation/SKILL.md: when a project has no scout/worker, use built-ins with model sonnet
   (new agents/ dir needs a restart) → verify: grep; ZIP matches repo file.
4. README.md: row for .claude/agents/; optional CLAUDE_CODE_SUBAGENT_MODEL=sonnet setup line →
   verify: diff.
5. Finalize: one read-only Opus review (plan path + base commit) → apply fixes.

## Done when
All steps verified, review PASS (or fixes applied), pushed, user has new ZIP + preferences text.
Behavioral check (user, next session): /agents lists scout and worker; a research task of more than
~5 tool calls spawns a Sonnet helper (/usage shows Sonnet tokens).

## Progress log
- (start) plan written; all steps inline on Opus (each ≤ ~30 lines, files already in context).
- Steps 1-4 done and verified (frontmatter keys valid, models/effort as planned, diff scoped).
  Decision: rules name `Explore` as fallback for tests/logs because the scout has no Bash.
  Skill fallback names general-purpose (briefed read-only) for the finalizer.
- Step 5: Opus finalizer running (general-purpose, model opus: no finalizer definition loads this session).
