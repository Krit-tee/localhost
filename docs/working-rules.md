Working Rules (apply to every task)

A. Iron Rules decide who works, on which model, at what cost. B. Coding Guidelines decide how code is written. C. Communication decides how you talk to me. Project CLAUDE.md may add specifics (agent names, plan folder, commands); it refines these rules, never loosens them.

A. Iron Rules — Models & Phases
Default: one Opus session leads the whole task (plan → execute → review). Sonnet and Haiku are sub-agent workers, never lead sessions.

Trivial exemption: pure questions/advice, or an obvious edit of ≤ ~10 lines in one file → just do it and say you're treating it as trivial. Everything else:

1. Plan — Opus. Explore only enough to plan; delegate broad searches to Explore (Haiku). Write a self-contained plan file (default docs/plans/<YYYY-MM-DD>-<slug>.md): goal, assumptions and open questions, files + line ranges, steps as 1. [step] → verify: [check] sized as sub-agent briefs, tests, done-criteria, base commit. Update CLAUDE.md only if commands/architecture change. If open questions remain, stop and ask; otherwise continue to Execute in the same session.
2. Execute — Opus leads, Sonnet assists. Opus implements coupled or tricky work itself and dispatches self-contained steps to Sonnet sub-agents (the plan step is the brief; its verify check is DONE WHEN). Opus reviews every worker diff before marking a step done. Fix small gaps yourself and note them in the plan file; ask me only if the goal must change.
   - Delegate to Sonnet when: the step is fully specified, touches ≤ ~3 files, and has an automated check.
   - Keep on Opus: design decisions, cross-cutting refactors, bugs with unclear cause, any step a worker already failed.
   - Escalation: a Sonnet worker fails its check twice → Opus takes the step back. No third retry.
3. Finalize — Opus review, once. When all steps pass, spawn one read-only Opus review agent with fresh context (plan path + base commit only), apply its fixes, re-run tests, report. Verdict FAIL → fix, then one "RE-REVIEW after FAIL". Hard cap: 2 Opus reviews per task; a second FAIL goes to me.

Long-task handoff (optional): if the context is getting heavy (e.g. after a compaction), write current progress into the plan file and suggest: "▶ open a NEW Opus session and say: continue <plan path>".

Sonnet 5.5 trial (until I remove this section): for every Sonnet worker, log one line in the plan file — step | passed first try (Y/N) | Opus rework (none/small/large). Include the tally in the final report.

Guards: check your own model first. Non-Opus session + non-trivial task → say so and ask whether to continue or switch. A session can't change its own model.

Delegation: before spawning ANY sub-agent, read ~/.claude/docs/delegation.md once per session and follow it (agent choice, brief template, budgets, report format). Changes of ≤ ~30 lines in files already in context: do them inline — an agent costs more.

B. Coding Guidelines (caution over speed; use judgment on trivial tasks)
B1. Think before coding. State assumptions (in Phase 1: in the plan). If several interpretations exist, present them rather than picking silently. If a simpler approach exists, say so and push back when warranted. If something is unclear, stop, name it, ask.

B2. Simplicity first. Minimum code that solves the problem. No features beyond the request, no single-use abstractions, no unrequested configurability, no error handling for impossible cases. If 200 lines could be 50, rewrite. Would a senior engineer call it overcomplicated?

B3. Surgical changes. Touch only what you must. Don't "improve" adjacent code, comments or formatting; don't refactor what isn't broken; match existing style. Mention unrelated dead code, don't delete it. Remove only the orphans your change created. Every changed line must trace to the request.

B4. Goal-driven execution. Turn tasks into verifiable goals: bug → reproducing test, then pass; validation → tests for invalid input, then pass; refactor → tests pass before and after. Loop until verified; each plan step's check is its worker's DONE WHEN.

C. Communication
Reply to me in Thai. Keep code, identifiers, commands, file paths, code comments, commit messages, plan files and sub-agent briefs in English. Technical terms may stay in English when a Thai translation would be unclear.

Working if: Opus leads and reviews every worker diff, Sonnet gets only well-scoped steps, diffs have no unrequested changes, and questions come before implementation rather than after mistakes.
