# Context-window management and handoff strategies in Claude Code: one long session vs sub-agents vs CLAUDE.md/plan file + /clear

Notes compiled 2026-09-29. Labels used: **[PRIMARY]** means verified at an Anthropic doc or post, or at the original study. **[SECONDARY]** means aggregator or search-snippet material. **[ANECDOTAL]** means a practitioner blog. **[OLDER]** marks pre-2026 material. Egress to research.trychroma.com, trychroma.com, arxiv.org, yage.ai and zenml.io was blocked, so Chroma study details come from secondary summaries plus the Chroma GitHub README. No arXiv paper could be read directly.

## Q1. Evidence on "context rot": does accuracy degrade as context grows, and at what fill level?

### Takeaway
Anthropic's current docs say plainly that accuracy and recall degrade as token count grows, and they treat context as the scarcest resource in Claude Code. No source gives a documented fill-percentage threshold. The Chroma study shows degradation on simple tasks and a very large focused-vs-full gap for Claude models. The often-quoted "degrades at 40–60% fill" figure comes from practitioners, not from measurement. No independent long-context benchmarks for Opus 5.5 or Sonnet 5.5 were found.

### Cited Findings
- **[PRIMARY, 2026 docs]** The platform docs say: "more context isn't automatically better. As token count grows, accuracy and recall degrade, a phenomenon known as *context rot*. This makes curating what's in context just as important as how much space is available." — [Claude API docs: Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- **[PRIMARY]** Opus 5.5, Opus 5, Opus 4.6–4.8, Sonnet 5.5, Sonnet 5 and Sonnet 4.6 all have a 1M-token context window, and 1M is the default. Long-context requests are billed at standard pricing. Sonnet 4.5 has 200K. — [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- **[PRIMARY, 2026 Claude Code docs]** "Most best practices are based on one constraint: Claude's context window fills up fast, and performance degrades as it fills… When the context window is getting full, Claude may start 'forgetting' earlier instructions or making more mistakes. The context window is the most important resource to manage." — [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY, 2025-09-29, OLDER]** Anthropic defines context rot this way: "as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases." It gives two causes. Transformers create "n² pairwise relationships for n tokens", which stretches the attention budget thin, and models see fewer long sequences in training. Its guiding principle is to find "the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome." — [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- **[PRIMARY, README only, July 2025, OLDER]** Chroma (Hong, Troynikov, Huber, 2025): "Model performance varies significantly as input length changes, even on simple tasks." The experiments cover NIAH extensions (semantic needles, distractors), LongMemEval, and a repeated-words replication task. — [chroma-core/context-rot GitHub](https://github.com/chroma-core/context-rot)
- **[SECONDARY, via search summaries of the Chroma report]** 18 models were tested, including Claude 4 (Opus 4, Sonnet 4), GPT-4.1, Gemini 2.5 and Qwen3. In LongMemEval, the focused prompt averaged ~300 tokens and the full prompt ~113K tokens, and every model family scored significantly higher on the focused prompt. Claude models showed the widest focused-vs-full gap, mostly because they abstained rather than hallucinated. The summaries characterise this as "Claude models abstain conservatively; GPTs hallucinate confidently." — [Chroma: Context Rot](https://www.trychroma.com/research/context-rot) (primary URL, blocked here; details via search-result summaries)
- **[SECONDARY]** Claude Opus 4.6 reportedly scored 76% on MRCR v2 8-needle at 1M tokens. — search summary citing [yage.ai long-context benchmarks (2026-03-15)](https://yage.ai/share/long-context-benchmark-en-20260315.html) (not verified; page blocked)
- **[PRIMARY, 2026-09-24]** The Opus 5.5 launch post reports on real Claude Code usage from March to September 2026. Claude "works 3.3x longer on each prompt with more than 40% more model calls per prompt", context per request grew "2.6x", and the input:output token ratio went "from 189:1 to 324:1". The post gives no long-context accuracy benchmark numbers. — [Claude blog: Opus 5.5 built for coding sessions that use more context](https://claude.com/blog/claude-opus-5-5-built-for-coding-sessions-that-use-more-context)
- **[PRIMARY, 2026-03-24]** Sonnet 4.5 showed "context anxiety": it began "wrapping up work prematurely as they approach what they believe is their context limit." The post says "Opus 4.5 largely removed that behavior on its own." — [Anthropic: Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- **[PRIMARY]** "Context awareness" means the API injects the token budget and remaining tokens. Sonnet 5, Sonnet 4.6, Sonnet 4.5 and Haiku 4.5 get this. Opus 4.7 and later and Sonnet 5.5 do **not** receive these tags; the docs point to task budgets (beta) instead. — [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- **[ANECDOTAL, Aug 2025, OLDER]** Dex Horthy (HumanLayer) recommends designing the whole workflow around keeping context utilization in the "40–60%" range ("frequent intentional compaction"). He ranks what can go into context from worst to least bad: incorrect information, then missing information, then noise. — [HumanLayer: Advanced context engineering for coding agents](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md)
- **[ANECDOTAL/SECONDARY]** Blogs describe a "dumb zone" that "kicks in around 40–60% of capacity", with "reasoning quality eroding since ~10–20% fill". — search summaries of [agentpatterns.ai: The Dumb Zone](https://learn.agentpatterns.ai/context-engineering/the-dumb-zone/) and [Duncan Leung blog](https://duncanleung.com/blog/claude-code-precompact-postcompact-context-management/). The same summaries say auto-compaction triggers "at ~95% fill". That contradicts the current docs, which say compaction runs at the context limit, or at ~967K for native-1M models (see Q2), so treat the 95% figure as outdated.

### Inferences
- The direction of the effect is well established: more tokens means lower recall and reasoning quality. The fill level where quality drops is not. Anthropic publishes no threshold, and the 40–60% rule of thumb is practitioner lore, most of it from the 200K-window era.
- With 1M windows now the default for Opus 5.5 and Sonnet 5.5, auto-compaction at ~967K means a session can run a very long time before any automatic intervention. Context-rot evidence suggests quality has degraded well before that point, so manual /clear, /compact or /autocompact matters more now than it did at 200K.
- Chroma's finding that Claude abstains under long, ambiguous contexts suggests that for Claude, long-context failure often looks like "can't find it / declines", not confident hallucination. That behaviour was measured on Claude 4 models and may differ on the 5.x generation.

### Gaps
- No measured long-context results (MRCR, GraphWalks, Fiction.LiveBench, LongMemEval) were found for Opus 5.5 or Sonnet 5.5 from Anthropic or any independent source reachable here.
- The Chroma technical report and arXiv papers (e.g., arXiv 2510.05381, "context length alone hurts performance") could not be read directly because egress was blocked, so their exact numbers are not verified here.
- No study measures Claude Code task accuracy as a function of context-fill percentage.

## Q2. How does Claude Code auto-compaction work now, what survives it, what is lost, and what is the official guidance on /clear, /compact and fresh sessions?

### Takeaway
Compaction replaces the whole conversation with a structured summary. Startup content (system prompt, project-root CLAUDE.md, unscoped rules, auto memory, the plan-mode plan) is re-injected from disk, up to five recent files are re-read, and invoked skills are restored with caps. Verbatim tool outputs, intermediate reasoning, path-scoped rules and anything said only in conversation are lost. Official guidance: /clear between unrelated tasks and after two failed corrections, /compact with a focus before a long new task, and a fresh session to execute a written spec.

### Cited Findings
- **[PRIMARY]** Default thresholds: without an override, Claude Code compacts at the model's context limit. Models with a native 1M window (Sonnet 5 and 5.5, Opus 4.7 and later on the Anthropic API, Fable) compact at "about 967K tokens by default" "to ensure quality and prevent hitting the hard limit mid-conversation". Sonnet 4.6 and Opus 4.6 without extended context, Opus 4.8 and later at 200K on Bedrock/Vertex/Foundry, and anything run with `CLAUDE_CODE_DISABLE_1M_CONTEXT=1` compact at the 200K boundary. — [Claude Code docs: Model config](https://code.claude.com/docs/en/model-config)
- **[PRIMARY]** Users can compact earlier. `/autocompact 500k` saves `autoCompactWindow`; there is also the `--autocompact` flag and the `CLAUDE_CODE_AUTO_COMPACT_WINDOW` env var, which takes precedence. `/autocompact auto` restores the model's tuned window. — [Model config](https://code.claude.com/docs/en/model-config); [Context window](https://code.claude.com/docs/en/context-window)
- **[PRIMARY]** What survives compaction (official table):
  - System prompt and output style still apply.
  - Project-root CLAUDE.md and unscoped rules are re-injected from disk.
  - Auto memory is re-injected.
  - A fresh git status is read.
  - "The plan Claude wrote in plan mode" is re-injected from disk.
  - Path-scoped rules and nested CLAUDE.md reload only when Claude next reads matching files.
  - Up to five of the most recently modified files Claude read or edited are re-read. A file over 5,000 tokens comes back only as a path reference.
  - Invoked skill bodies are re-injected, capped at 5,000 tokens per skill and 25,000 in total, oldest dropped first.
  - Background commands and subagents keep running.
  - "Context that hooks added earlier" is summarized away.
  - SessionStart hooks matching `compact` run again.
  - The skill listing (descriptions) is **not** re-injected.

  — [Claude Code docs: Explore the context window](https://code.claude.com/docs/en/context-window)
- **[PRIMARY]** The summary "keeps: your requests and intent, key technical concepts, files examined or modified with important code snippets, errors and how they were fixed, pending tasks, and current work. It replaces the verbatim conversation: full tool outputs and intermediate reasoning are gone… Claude can still reference the rest of the work but won't have the exact content." Since v2.1.198 the summarization request inherits the session's extended-thinking setting. — [Context window](https://code.claude.com/docs/en/context-window)
- **[PRIMARY]** "If an instruction disappeared after compaction, it was given only in conversation, lives in a nested CLAUDE.md that hasn't reloaded yet, or is a path-scoped rule that hasn't matched a file since. Add conversation-only instructions to CLAUDE.md to make them persist." — [Claude Code docs: Memory](https://code.claude.com/docs/en/memory)
- **[PRIMARY]** Controls:
  - `/compact <instructions>`, e.g. "/compact Focus on the API changes".
  - Partial compaction through `/rewind`, then "Summarize from here" or "Summarize up to here".
  - A "Compact instructions" section in CLAUDE.md, e.g. "When compacting, always preserve the full list of modified files and any test commands".
  - `/btw` for side questions that never enter history.

  — [Best practices](https://code.claude.com/docs/en/best-practices); [Costs](https://code.claude.com/docs/en/costs)
- **[PRIMARY]** When to /clear:
  - "Use `/clear` frequently between tasks to reset the context window entirely."
  - "If you've corrected Claude more than twice on the same issue in one session, the context is cluttered with failed approaches. Run `/clear` and start fresh with a more specific prompt… A clean session with a better prompt almost always outperforms a long session with accumulated corrections."
  - Named anti-patterns: "kitchen sink session", "correcting over and over", "infinite exploration".

  — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY]** When to start a fresh session: after the interview-and-spec step, "Once the spec is complete, start a fresh session to execute it. The new session has clean context focused entirely on implementation, and you have a written spec to reference." Also: "A fresh context improves code review since Claude won't be biased toward code it just wrote." — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY]** The docs also give the counterpoint: "Sometimes you *should* let context accumulate because you're deep in one complex problem and the history is valuable." — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY]** Cost facts:
  - "`/compact` reads the conversation it summarizes, so compacting a large context is itself a large request. When you want a fresh start instead of continuity, `/clear` costs nothing."
  - "Stale context wastes tokens on every subsequent message."
  - "Unexpectedly high spend… usually traces back to long sessions that were never cleared or to Opus left as the default model."
  - A one-line question in a long session "still draws usage for the whole conversation" (at cache-read rates).
  - Cache lifetime is 1h on subscriptions, 5 min on API by default. A first message after a longer break misses the cache and reprocesses the full context.

  — [Claude Code docs: Manage costs](https://code.claude.com/docs/en/costs)
- **[PRIMARY, 2026-09-24]** Opus 5.5 guidance: "Pick your model at the start of a session rather than switching midway"; "Compact before you step away rather than after"; set a one-hour cache lifetime for long sessions. Cached-read price dropped 60%. — [Claude blog: Opus 5.5](https://claude.com/blog/claude-opus-5-5-built-for-coding-sessions-that-use-more-context)
- **[PRIMARY]** Enterprise average is about $13 per developer per active day and $150–250 per month; 90% of users stay under $30 per active day. — [Costs](https://code.claude.com/docs/en/costs)

### Inferences
- Anything that must persist through compaction belongs on disk, in project-root CLAUDE.md, an unscoped rule, auto memory, or the plan-mode plan file. These are the only channels guaranteed to be re-injected. Instructions given in conversation, rules loaded through hooks, and path-scoped rules are the main loss vectors.
- /clear plus a handoff file costs less than /compact: /clear is free, and /compact is a full-context request. /compact keeps continuity at the cost of an automatic summary whose contents you only partly control.
- "Compact before stepping away" matters because of the cache TTL. After an idle gap, the first request reprocesses the whole uncompacted context at full input price.

### Gaps
- No official statement gives a recommended fill percentage for manual /compact or /clear. The docs say "when context starts affecting performance or before a long new task."
- No official measurement compares task quality after auto-compaction with quality after /clear plus a handoff doc.

## Q3. What is official guidance on sub-agents for keeping the main context clean, what do they cost, how do results return, and what are the information-loss risks?

### Takeaway
Anthropic recommends sub-agents mainly as a context-isolation tool: verbose exploration, test and log output, and review stay in the sub-agent's window, and only a summary returns. They are not free. Each spends its own tokens, many detailed returns still bloat the main context, a non-fork sub-agent starts with no conversation history, and multi-agent systems are a poor fit for tightly coupled coding work.

### Cited Findings
- **[PRIMARY]** "Since context is your fundamental constraint, use subagents to keep research out of it… Subagents run in separate context windows and report back summaries." — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY]** "The verbose output stays in the subagent's context while only the relevant summary returns to your main conversation." In the context-window walkthrough, "Only the summary and a small metadata trailer come back." — [Sub-agents docs](https://code.claude.com/docs/en/sub-agents); [Context window](https://code.claude.com/docs/en/context-window)
- **[PRIMARY]** Cost caveats:
  - Each sub-agent "sends its own requests, which count toward the same usage limits as your main conversation."
  - "Running many subagents that each return detailed results can consume significant context, and each subagent spends tokens of its own while it runs."
  - Mitigation: "choose a smaller model for a subagent" (e.g. `model: haiku`), or force one model with `CLAUDE_CODE_SUBAGENT_MODEL` plus `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` (v2.1.257+).

  — [Sub-agents](https://code.claude.com/docs/en/sub-agents); [Costs](https://code.claude.com/docs/en/costs)
- **[PRIMARY]** What a non-fork sub-agent sees:
  - It gets its own system prompt, the task message Claude writes, all CLAUDE.md levels (except the built-in Explore and Plan agents, which skip CLAUDE.md), a git status snapshot, and preloaded skills.
  - It does **not** get conversation history, the main session's auto memory, or the output style.
  - A **fork** inherits the entire conversation, system prompt, tools and model.

  — [Sub-agents](https://code.claude.com/docs/en/sub-agents); [Memory](https://code.claude.com/docs/en/memory)
- **[PRIMARY]** "Forked subagents start from the parent's cache" (Opus 5.5 launch), which lowers the cost of forks. — [Claude blog: Opus 5.5](https://claude.com/blog/claude-opus-5-5-built-for-coding-sessions-that-use-more-context)
- **[PRIMARY]** When to use the main conversation instead:
  - "The task needs frequent back-and-forth or iterative refinement."
  - "Multiple phases share significant context (planning → implementation → testing)."
  - "Quick, targeted change."
  - "Latency matters."

  Use sub-agents when "the task produces verbose output you don't need in main context… the work is self-contained and can return a summary." — [Sub-agents](https://code.claude.com/docs/en/sub-agents)
- **[PRIMARY]** Sub-agents auto-compact with the same logic, and `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` applies to them. They can be resumed with full history through SendMessage; the built-in Explore and Plan agents are one-shot. Transcripts are stored separately and main-session compaction does not touch them. Default nesting depth is 3 layers, and at most 20 sub-agents can run concurrently. — [Sub-agents](https://code.claude.com/docs/en/sub-agents)
- **[PRIMARY]** Adversarial review in a fresh sub-agent: the reviewer "sees only the diff and the criteria you give it, not the reasoning that produced the change." The docs warn that reviewers asked to find gaps will usually find some, which leads to over-engineering, so tell them to flag only correctness or requirement gaps. — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY, 2025-09-29, OLDER]** Sub-agents return a "condensed, distilled summary of its work (often 1,000–2,000 tokens)". Anthropic's selection guide: compaction for extensive back-and-forth, note-taking for iterative development with clear milestones, multi-agent for complex research needing parallel exploration. — [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- **[PRIMARY, 2025-06-13, OLDER]** Anthropic's multi-agent research system:
  - Agents use ~4× the tokens of chat, and multi-agent systems ~15×.
  - Token usage explained 80% of performance variance on BrowseComp.
  - Opus 4 lead with Sonnet 4 sub-agents scored 90.2% better than single-agent Opus 4 on internal research evals.
  - Multi-agent is a poor fit for "most coding tasks", for domains "requiring shared context across all agents", and where there are many interdependencies.
  - Sub-agents write artifacts to the filesystem to avoid information loss (the "game of telephone").
  - Delegation prompts need an objective, an output format, tool guidance and clear boundaries; vague delegation caused duplicated work and gaps.

  — [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- **[PRIMARY]** Agent teams use "approximately 7x more tokens than standard sessions when teammates run in plan mode." — [Costs](https://code.claude.com/docs/en/costs)
- **[ANECDOTAL, 2025, OLDER]** "Subagents are not about playing house and anthropomorphizing roles. Subagents are about context control." — [HumanLayer ACE-FCA](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md)

### Inferences
- Information-loss risk comes from the summary bottleneck. The main agent sees only what the sub-agent chose to report, and a non-fork sub-agent sees only what the delegation prompt said, with no conversation history. Mitigations: write specific delegation prompts (objective, output format, boundaries), have the sub-agent write full findings to a file and return a pointer plus summary, and use a fork when the sub-agent needs the conversation.
- On cost, a sub-agent is cheaper for the main session, because it avoids re-sending thousands of tokens of exploration on every later turn. Total spend is higher, since the sub-agent pays for its own window. Routing sub-agents to Sonnet or Haiku narrows the gap.
- For tightly coupled implementation work (plan → code → test sharing a lot of state), the docs and the multi-agent post both favour the main conversation, or a single session with notes, over fanning work out.

### Gaps
- No official numbers were found for the token overhead of a typical Claude Code sub-agent call (as opposed to the 15× figure for research-system multi-agent runs, or 7× for agent teams).
- No measurement was found of accuracy loss from sub-agent summarization in coding tasks.

## Q4. What are the best practices for handoff (plan, progress and NOTES files, structured note-taking), what does a good handoff doc contain, and how does CLAUDE.md size affect adherence?

### Takeaway
Anthropic's long-running-agent work converges on external, structured state: a progress log, a machine-readable feature/task list (JSON), git commits, and an init script. Each new session starts with a fixed "get your bearings" routine. Keep CLAUDE.md under 200 lines of broadly applicable rules, and put session state in plan or progress files, not in CLAUDE.md.

### Cited Findings
- **[PRIMARY, 2025-11-26, OLDER]** The long-running agent harness:
  - An initializer agent creates `init.sh`, a `claude-progress.txt` log and an initial git commit, plus a JSON feature list of 200+ features marked passing or failing.
  - Coding agents may only flip the `passes` field ("It is unacceptable to remove or edit tests").
  - JSON was chosen because "the model is less likely to inappropriately change or overwrite JSON files compared to Markdown files."
  - Each session starts with `pwd`, reads the git log and progress file, and picks the highest-priority incomplete feature.
  - Work proceeds one feature at a time, with a commit and a progress update at the end.
  - Compaction alone was insufficient: Opus 4.5 "in a loop across multiple context windows will fall short of building a production-quality web app if it's only given a high-level prompt."
  - Failure modes named: premature victory, undocumented environment, features marked done too early, confusion about how to run the app.

  — [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- **[PRIMARY, 2026-03-24]** Context resets with a structured handoff artifact (carrying "the previous agent's state and the next steps") were essential for Sonnet 4.5, despite the extra tokens and latency. Opus 4.5 and 4.6 later made resets unnecessary: the final harness ran "as one continuous session" with automatic compaction via the Agent SDK. Agents communicated through files. Cost examples: 6 hours for $200 with the full harness, and 3h50m for $124.70 with the updated harness. — [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- **[PRIMARY, 2025-09-29, OLDER]** Structured note-taking means persistent external notes (NOTES.md, to-do lists). In the Pokémon example, an agent kept strategic notes and maps across thousands of steps and context resets. "Just-in-time" retrieval means keeping file paths and queries as references and loading content on demand. — [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- **[PRIMARY]** "For agents that span multiple sessions, design your state artifacts so that context recovery is fast when a new session starts" (points to the memory tool's multisession pattern). — [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- **[PRIMARY]** A good spec or handoff for a fresh session is "self-contained: they name the files and interfaces involved, state what is out of scope, and end with an end-to-end verification step that proves the feature works." — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY]** The plan written in plan mode is re-injected from disk after compaction. — [Context window](https://code.claude.com/docs/en/context-window)
- **[ANECDOTAL]** A compaction/progress artifact should hold the end goal, the approach, the steps completed, and the current blockers or failures. — [HumanLayer ACE-FCA](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md)
- **[PRIMARY]** CLAUDE.md size:
  - "target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence."
  - `@imports` "still load… at launch", so they don't reduce context.
  - Use path-scoped rules or skills for content that is only sometimes relevant.
  - A warning is shown when a file exceeds the recommended length or when files together exceed a combined limit.
  - Files over 4 MiB are skipped.
  - CLAUDE.md is delivered "as a user message after the system prompt", with no guarantee of strict compliance.

  — [Memory](https://code.claude.com/docs/en/memory)
- **[PRIMARY]** "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" For each line, ask "Would removing this cause Claude to make mistakes?" Exclude "Information that changes frequently." — [Best practices](https://code.claude.com/docs/en/best-practices)
- **[PRIMARY]** Auto memory loads only the first 200 lines or 25KB of `MEMORY.md`. Topic files are read on demand, and Claude Code warns when MEMORY.md nears the limit. Auto memory does not load in non-fork sub-agents. — [Memory](https://code.claude.com/docs/en/memory)
- **[PRIMARY]** Rules that must hold every time belong in hooks, not CLAUDE.md, because hooks are deterministic and CLAUDE.md is advisory. A SessionStart hook with the `compact` matcher can re-inject custom context after compaction. — [Best practices](https://code.claude.com/docs/en/best-practices); [Context window](https://code.claude.com/docs/en/context-window)

### Inferences
- A good handoff doc, synthesized from Anthropic and practitioner sources, contains:
  1. The goal and acceptance criteria / verification command.
  2. Scope and out-of-scope.
  3. Files and interfaces involved.
  4. Decisions made and why.
  5. Done so far, tied to commits.
  6. Current state, blockers and failed approaches, so they are not retried.
  7. Next steps in priority order.
  8. How to run and test (init script).

  A machine-readable task list (JSON with pass/fail) resists accidental edits better than Markdown prose.
- Session state should go into a separate plan or progress file (e.g. PLAN.md or claude-progress.txt), not into CLAUDE.md. CLAUDE.md should hold stable rules, and the docs say to exclude "information that changes frequently." Putting progress into CLAUDE.md would inflate a file that loads every session and hurt adherence.
- To make a plan file survive auto-compaction, either use the plan-mode plan, which is re-injected, or @-import it from CLAUDE.md. The import costs context in every session, so it should be temporary.

### Gaps
- No official template exists for a Claude Code handoff file beyond the spec guidance and the harness posts.
- No quantitative data links CLAUDE.md line count to adherence rates. The 200-line figure is guidance, not a published measurement.

## Q5. Is "update CLAUDE.md or a plan file, then /clear and start a new session" better than continuing one long session with heavy sub-agent use?

### Takeaway
No controlled measurement settles this. The weight of official guidance and practitioner consensus supports a hybrid: one focused session per task or phase, sub-agents for verbose research and review, and at natural boundaries (spec done, phase done, repeated failed corrections, unrelated task) write state to a plan or progress file and /clear. Anthropic's own 2026 harness evidence shows that with Opus 4.5 and later, a single long session with auto-compaction can work for multi-hour builds. Resets are therefore not strictly required, but they stay cheaper and cleaner at task boundaries.

### Cited Findings
- **[PRIMARY]** For cleared or fresh sessions:
  - "A clean session with a better prompt almost always outperforms a long session with accumulated corrections."
  - After the spec is written, "start a fresh session to execute it."
  - `/clear` "costs nothing" while `/compact` is a large request.
  - "Long sessions with irrelevant context can reduce performance."

  — [Best practices](https://code.claude.com/docs/en/best-practices); [Costs](https://code.claude.com/docs/en/costs)
- **[PRIMARY]** For continuing a session: "Sometimes you *should* let context accumulate because you're deep in one complex problem and the history is valuable." Sub-agent docs favour the main conversation when "multiple phases share significant context (planning → implementation → testing)." Sessions can be named with `/rename`, treated "like branches", and resumed later. — [Best practices](https://code.claude.com/docs/en/best-practices); [Sub-agents](https://code.claude.com/docs/en/sub-agents)
- **[PRIMARY, 2026-03-24]** Context resets beat compaction for Sonnet 4.5 because compaction left "context anxiety" unresolved. With Opus 4.5 and 4.6, Anthropic "dropped context resets entirely" and ran one continuous session with automatic compaction. — [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- **[PRIMARY, 2025-11-26, OLDER]** Across multiple context windows, compaction alone was insufficient. Structured external state (progress file, JSON feature list, git) was needed for multi-session success. — [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- **[PRIMARY, 2025, OLDER]** Anthropic's heuristic: compaction for conversational flow, note-taking for iterative milestone work, and multi-agent for parallel research. Multi-agent is a poor fit for most coding. — [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents); [Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- **[PRIMARY, 2026-09-24]** Opus 5.5 is explicitly tuned and priced for longer, higher-context sessions: cache reads are 60% cheaper, cache misses fell by more than 50%, and forked sub-agents reuse the parent cache. This lowers the cost of continuing long sessions. — [Claude blog: Opus 5.5](https://claude.com/blog/claude-opus-5-5-built-for-coding-sessions-that-use-more-context)
- **[ANECDOTAL, 2025, OLDER]** Practitioner consensus (HumanLayer "frequent intentional compaction"): run research → plan → implement, write progress to a file and restart or compact at each phase, keep utilization at 40–60%, and use sub-agents purely for context control. Claimed results include fixing a bug in a 300K-LOC Rust codebase (BAML) and shipping 35K LOC in 7 hours with 2 engineers. These are self-reported. — [HumanLayer ACE-FCA](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md)

### Inferences
- A practical decision rule, synthesized from the sources above:
  - **Stay in the session** while working on one coupled problem whose history is valuable and there have been no repeated failures. Offload verbose reads, test runs and reviews to sub-agents, and use a focused `/compact`, or `/autocompact` set well below 967K, for long stretches.
  - **Write state to a file and /clear** when:
    - switching to an unrelated task;
    - after more than two failed corrections;
    - after a spec or plan is finalized, before implementation;
    - after a phase or feature completes;
    - before stepping away past the cache TTL.
  - **Use a fresh sub-agent or session for review**, so the reviewer isn't biased by the author's reasoning.
- On accuracy, writing state and clearing gives a smaller, curated context, which fits the "smallest set of high-signal tokens" principle. Its risk is loss of implicit knowledge that the handoff doc doesn't capture. A sub-agent-heavy long session keeps implicit history but accumulates summaries and decisions, and it depends on the lossy auto-summary once it compacts.
- On cost, /clear with a short handoff is the cheapest per turn and in total. Long sessions pay cache-read tokens on the whole history every turn, plus a full-context request at each compaction, plus sub-agent tokens. Opus 5.5 cache pricing narrows the gap but does not remove it.
- The "Opus-solo vs Sonnet sub-agents" angle: the 90.2% multi-agent gain (Opus lead with Sonnet sub-agents) was measured on breadth-first research, not coding. For coding, the docs suggest Sonnet or Haiku sub-agents for cheap isolated reads or tests, with the coupled implementation kept in one main session.

### Gaps
- No controlled experiment or published measurement compares, for Claude Code coding tasks, (a) one long session with auto-compaction, (b) sub-agent-heavy long sessions, and (c) handoff file plus /clear, on accuracy or cost. The recommendation rests on official guidance and anecdote.
- Anthropic's 2026 harness evidence (continuous session viable with Opus 4.5 and 4.6) comes from an autonomous harness with planner/generator/evaluator agents, not from interactive Claude Code use. How it transfers to Opus 5.5 or Sonnet 5.5 in interactive sessions is unmeasured.
- No 2026 practitioner survey or benchmark of these strategies was found. Most practitioner material predates the 1M-default era (2025, 200K windows).
