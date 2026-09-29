# Token cost and subscription-limit impact: one long Opus 5.5 session vs Opus 5.5 orchestrator + Sonnet 5.5 / Haiku 4.5 sub-agents

Research date: 2026-09-29. Legend: **[PRIMARY]** = verified by fetching the official page in this session. **[SECONDARY]** = came from a third-party page or a search-engine snippet. I could not open the underlying page for most of these (artificialanalysis.ai, kingy.ai, orcarouter.ai, the-decoder.com, tokencost.app, beri.net and systima.ai were all blocked by the network egress proxy). **[ANECDOTAL]** = developer blog measurement. **[OLDER MODEL]** = based on pre-5.5 model pricing or behavior.

## 1. Current prices and how caching / TTL changes the effective Opus-vs-Sonnet gap

### Takeaway
Opus 5.5 lists at exactly 2x Sonnet 5.5 for input, output and cache writes ($4/$20 vs $2/$10). Both models charge the **same $0.20/MTok for cache reads**, though, because Opus 5.5 gets a special 0.05x cache-read multiplier. In a long, cache-heavy Claude Code session the per-token gap therefore shrinks toward 1x on input and stays 2x only on output, cache writes and misses. Sub-agents work against this. They cannot read the parent's cache, and they default to a 5-minute TTL, so each one pays full cache-write rates to build its own prefix.

### Cited Findings
- **List prices per MTok [PRIMARY]** — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing):

  | Model | Base input | 5m cache write | 1h cache write | Cache hit/refresh | Output | Batch in/out |
  |---|---|---|---|---|---|---|
  | Claude Opus 5.5 | $4 | $5 | $8 | **$0.20** (0.05x) | $20 | $2 / $10 |
  | Claude Sonnet 5.5 | $2 | $2.50 | $4 | **$0.20** (0.1x) | $10 | $1 / $5 |
  | Claude Haiku 4.5 | $1 | $1.25 | $2 | $0.10 | $5 | $0.50 / $2.50 |
  | (ref) Opus 5 / 4.8 / 4.7 | $5 | $6.25 | $10 | $0.50 | $25 | |
  | (ref) Sonnet 5 | $2 | $2.50 | $4 | $0.20 | $10 | |
  | (ref) Sonnet 4.6 / 4.5 | $3 | $3.75 | $6 | $0.30 | $15 | |
- "Cache hits and refreshes on Claude Opus 5.5 are priced at 0.05x the base input price"; all other models except Fable/Mythos 5.1 use 0.1x — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) [PRIMARY]
- Cache multipliers: 5-minute write = 1.25x base input and 1-hour write = 2x. A 5m cache "pays off after one cache read", a 1h cache "after two cache reads". The multipliers stack with batch and data residency — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) [PRIMARY]
- Sonnet 5 pricing was introductory $2/$10 and "is now the standard price"; the planned rise to $3/$15 on Sept 1, 2026 "will not occur". Sonnet 5.5 lists at the same $2/$10 — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) [PRIMARY]
- Tokenizer: "Claude 4.7 and later models ... use a newer tokenizer ... approximately 30% more tokens for the same text", while "Claude Sonnet 4.6 and earlier models use the previous tokenizer". Haiku 4.5 predates 4.7, so it uses the old tokenizer — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) [PRIMARY]
- Long context: "Claude 4.6 and later models ... include the full 1M token context window at standard pricing". Opus 5.5 and Sonnet 5.5 have no >200K surcharge — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) [PRIMARY]
- Fast mode for Opus 5.5 costs $8 input / $40 output (2x). Turning it on causes a one-time full cache miss billed at fast-mode rates — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing); [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]
- The tool-use system prompt overhead is 286 tokens for both Opus 5.5 and Sonnet 5.5, and 496 for Haiku 4.5 — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing) [PRIMARY]
- **Claude Code TTL buckets [PRIMARY]** — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching):
  - On a Claude subscription within plan usage, the **main conversation gets a 1-hour TTL**. "Everything else" gets **5 minutes**: subagents, workflows, in-process teammates, forks, compaction and session titles, except some server-controlled helpers.
  - With usage credits, an API key or a cloud provider, both buckets default to 5 minutes.
  - You can override the TTL with `promptCacheTtl` / `CLAUDE_CODE_PROMPT_CACHE_TTL` for the main conversation, `subagentPromptCacheTtl` / `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` for subagents, a per-subagent `experimental.cacheTtl: 1h`, or `ENABLE_PROMPT_CACHING_1H=1`. A `1h` subagent setting is ignored while a subscription is drawing on usage credits.
- "A subagent starts its own conversation with its own system prompt and tool set ... Its first request doesn't read the parent's cache, because the two prefixes differ, and it warms a cache of its own across its turns. Subagents ... get five minutes even on a subscription." A **fork**, by contrast, "inherits the parent's system prompt, tools, and conversation history exactly, so its first request reads the parent's cache" — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]
- "Each model has its own cache. Switching models recomputes the entire request". `opusplan` switches between Opus in plan mode and Sonnet in execution, so "each plan-mode toggle is a model switch and starts a fresh cache" — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]
- On Opus 5.5 and Sonnet 5.5 with an API key or Claude subscription, "changing effort keeps the cache". This does not hold on Bedrock or Google Cloud, through the Claude apps gateway, or with HIPAA configurations — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]

### Inferences
- **Effective gap by token type (Opus 5.5 ÷ Sonnet 5.5):** output 2.0x, uncached input 2.0x, cache writes 2.0x, **cache reads 1.0x**. Claude Code sessions are usually dominated by cache reads by volume; the docs' own `/usage` example shows 940k cache read vs 50k cache write vs 1.2k input vs 5.3k output. In such a session the blended price gap is closer to the output share than to 2x. The number that really separates the models is output tokens, including thinking tokens.
- **Haiku 4.5 vs Opus 5.5:** 4x cheaper on input, output and writes, but only 2x cheaper on cache reads ($0.10 vs $0.20). Haiku also uses the older tokenizer, which is about 30% fewer tokens for the same text, so its real per-text advantage is somewhat larger than the list ratio.
- **The TTL asymmetry hurts sub-agents.** A subscription's main Opus thread re-reads its prefix at $0.20/MTok for up to an hour between turns. A sub-agent that idles more than 5 minutes, for example while waiting on a long test run, rebuilds its prefix at write rates: $2.50/MTok on Sonnet 5.5, $1.25 on Haiku 4.5.
- **A worked example of the first-request penalty (illustrative arithmetic from the list prices).** Take a sub-agent whose opening prompt is 20K tokens (system prompt, CLAUDE.md, git status, task).
  - Its first request costs 20K × $2.50 = $0.05 on Sonnet 5.5 (5m write) and $0.025 on Haiku 4.5.
  - Keeping the same 20K inside a warm Opus main thread costs 20K × $0.20 = $0.004 per re-read.
  - The sub-agent only wins if what it keeps *out* of the main thread is larger than what it costs to set up.
- Forks keep the parent's model and cache, so they are cheap to start. They cannot run on a cheaper model, though, because a different model means a different cache.

### Gaps
- I did not verify whether the Opus 5.5 0.05x cache-read discount also applies on Bedrock or Google Cloud. The pricing page defers to those providers' own pricing pages.
- I found no official ratio of cache-read to output token volume for a typical Claude Code task on 5.5-generation models.

## 2. Measured tokens-per-task and cost-per-task by effort level; where Sonnet stops being cheaper

### Takeaway
Per Artificial Analysis figures relayed by secondary sources, Sonnet 5.5 is cheaper per task than Opus 5.5 **at the same effort setting** only from low to xhigh, by roughly 21–56%. **At max effort Sonnet 5.5 costs more per task** than Opus 5.5 ($7.60 vs $5.98 per Intelligence Index task). Compared at **equal intelligence score**, Opus 5.5 is about as cheap or cheaper at every data point found: score about 42 costs $0.55 on Opus low vs $0.59 on Sonnet medium, and score 56 costs $3.46 on Opus xhigh vs $7.60 on Sonnet max. Sonnet 5.5's price advantage is eaten by its much higher output-token count at high effort.

### Cited Findings
- AA Intelligence Index, "max with fallback" setting: Opus 5.5 scores 58 and Sonnet 5.5 scores 56 — [Artificial Analysis search snippet](https://artificialanalysis.ai/models/claude-opus-5-5); [AA article: Sonnet 5.5 reaches #2](https://artificialanalysis.ai/articles/claude-sonnet-5-5) [SECONDARY: AA pages blocked, figures from search snippets]
- AA's launch post: Sonnet 5.5 "scores 56 on the Artificial Analysis Intelligence Index, just 2 points behind Opus 5.5 (max), but at the highest Output Tokens per Task we've seen". With max effort, Sonnet 5.5 "gains 18 points over Sonnet 5" — [Artificial Analysis on X](https://x.com/ArtificialAnlys/status/2104640155843989864) [SECONDARY: tweet title via search snippet]
- At max effort Sonnet 5.5 used about **193k output tokens per Intelligence Index task**, "about 60% higher than Opus 5.5 (max) or Sonnet 5 (max)". The totals were **410M output tokens** for Sonnet 5.5's index run vs **260M** for Opus 5.5's — [AA article](https://artificialanalysis.ai/articles/claude-sonnet-5-5) via search snippet [SECONDARY]
- **Cost per AA Intelligence Index task by effort**, collated from AA data by secondary sites ([digitalapplied](https://www.digitalapplied.com/blog/sonnet-5-5-or-opus-5-5-effort-level-cost-per-task), [beri.net](https://www.beri.net/article/claude-sonnet-5-5-vs-opus-5-5-cost-per-task-matched-score-effort-between-tools-migration), [tokencost](https://tokencost.app/blog/claude-sonnet-5-5-pricing), [explainx](https://www.explainx.ai/blog/claude-opus-5-5-vs-sonnet-5-5-comparison-2026)) [SECONDARY, not verified at AA]:

  | Effort | Opus 5.5 score / $ per task | Sonnet 5.5 score / $ per task |
  |---|---|---|
  | low | 42 / $0.55 | not found |
  | medium | not found | 41 / $0.59 |
  | high | 54 / $1.82 | not found |
  | xhigh | 56 / $3.46 | 52 / $2.74 |
  | max (with fallback) | 58 / $5.98 | 56 / $7.60 |
- "From low to xhigh, Sonnet 5.5 costs less per task than Opus 5.5 at the same setting, by 21 to 56 per cent. At max the order flips: $10.67 against $8.40" — search snippet from [digitalapplied](https://www.digitalapplied.com/blog/sonnet-5-5-or-opus-5-5-effort-level-cost-per-task) / [aivy](https://aivy.com.au/resources/claude-sonnet-5-5-vs-opus-5-5/) [SECONDARY]. **Conflict:** the $10.67 vs $8.40 max-effort pair does not match AA's $7.60 vs $5.98. It probably comes from a different benchmark or cost basis, possibly Anthropic's launch chart, but I could not confirm which. Both pairs agree on the direction: Sonnet costs more at max.
- The press headline "costing up to 30 percent less per task" for Sonnet 5.5 vs Opus 5.5 ([the-decoder](https://the-decoder.com/anthropics-claude-sonnet-5-5-nearly-matches-opus-5-5-on-benchmarks-while-costing-up-to-30-percent-less-per-task/)) is framed by [orcarouter](https://www.orcarouter.ai/blog/claude-sonnet-5-5-cost-per-task) as "The 30% Cost Claim vs $7.60 Per Task" [SECONDARY; pages blocked, titles only]. The claim is presumably Anthropic's, but I could not see its benchmark or effort setting.
- Default effort in Claude Code: "`high` on every model that supports effort, except that Opus 5.5 and Sonnet 5.5 default to `medium`". All five levels (low/medium/high/xhigh/max) are available on both — [Claude Code model config](https://code.claude.com/docs/en/model-config) [PRIMARY]. One secondary snippet says Sonnet 5.5's **API** default is `high` — [search snippet](https://www.digitalapplied.com/blog/sonnet-5-5-or-opus-5-5-effort-level-cost-per-task) [SECONDARY]
- "Thinking tokens are billed as output tokens ... You can't turn off thinking on Opus 5.5, Sonnet 5.5, or the Fable models, which always use extended thinking." Lowering effort is the lever — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]

### Inferences
- **Crossover by setting vs by score.** At the same effort setting, Sonnet stops being cheaper between xhigh and max. Judged at the same capability, Sonnet 5.5 is never clearly cheaper in the AA data. Opus 5.5 low roughly matches Sonnet 5.5 medium on both score and cost, and Opus high (54, $1.82) beats Sonnet xhigh (52, $2.74) on both.
- On the AA mix, **"Opus 5.5 at a lower effort" usually beats "Sonnet 5.5 at a higher effort"** for the same quality. Sonnet sub-agents pay off mainly on **easy, low-effort, input-heavy work**: search, file reading, log triage and test running, where output tokens are small and Sonnet's 2x cheaper input and write rates dominate.
- AA's Intelligence Index is a reasoning and knowledge mix, not an agentic coding workload. Cost per task in a Claude Code session also depends heavily on cache reads, which AA's per-task cost may not reflect in the same way.
- Claude Code's `medium` default for both 5.5 models puts typical usage in the range where Sonnet is modestly cheaper per task at the same setting.

### Gaps
- I could not get AA's per-task numbers for Opus 5.5 medium, Sonnet 5.5 low and high, or output tokens per task at each effort level, because AA is blocked by the proxy.
- I could not retrieve Anthropic's own Sonnet 5.5 launch post: anthropic.com/news/claude-sonnet-5-5 returned 404, so its chart numbers are unverified.
- I found no independent measurement of per-effort cost on SWE-bench-style agentic coding tasks for these two models.

## 3. Sub-agent overhead, multipliers, built-in agents' models, and measured savings

### Takeaway
Every sub-agent is a separate conversation that pays its own preamble and its own cache warm-up. Official multipliers are about **7x tokens for agent teams in plan mode** vs a standard session, and **about 15x vs chat for Anthropic's multi-agent research system**; plain agents use about 4x vs chat. Crucially, built-in **Explore and Plan inherit the session model**, so on an Opus 5.5 session they run Opus 5.5 and save nothing on price. You only get Sonnet or Haiku pricing by setting it explicitly.

### Cited Findings
- "Agent teams use approximately 7x more tokens than standard sessions when teammates run in plan mode, because each teammate maintains its own context window and runs as a separate Claude instance." Advice: "Use Sonnet for teammates", "Keep teams small", and "token usage is roughly proportional to team size" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- "agents typically use about 4× more tokens than chat interactions, and multi-agent systems use about 15× more tokens as chats"; "token usage by itself explains 80% of the variance" in performance; "multi-agent system with Claude Opus 4 as the lead agent and Claude Sonnet 4 subagents outperformed single-agent Claude Opus 4 by 90.2%" — [Anthropic engineering: multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) [PRIMARY, OLDER MODEL: Opus 4 / Sonnet 4, research workload, not coding]
- Built-in agent models: "As of v2.1.198, Explore inherits the main conversation's model instead of always running on Haiku. On the Claude API, the inherited model is capped at Opus". Plan also inherits. General-purpose follows the resolution order: per-invocation `model` → subagent frontmatter `model` (with `inherit`) → `CLAUDE_CODE_SUBAGENT_MODEL` → main conversation model — [Claude Code sub-agents](https://code.claude.com/docs/en/sub-agents) [PRIMARY]
- "Setting `CLAUDE_CODE_SUBAGENT_MODEL` by itself doesn't change the model the built-in Explore and Plan subagents run on." You also need `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` — [Claude Code sub-agents](https://code.claude.com/docs/en/sub-agents) [PRIMARY]
- "A switch to Opus also applies to the subagents that inherit your session's model. For simple subagent tasks, specify `model: haiku`" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- What a non-fork subagent's context starts with: its own system prompt (not the Claude Code system prompt), the task message, the full CLAUDE.md hierarchy (**Explore and Plan skip it**), a git status snapshot, preloaded skills and a sibling roster. "Only the subagent's final summary returns to the main conversation" — [Claude Code sub-agents](https://code.claude.com/docs/en/sub-agents) [PRIMARY]
- "Delegate verbose operations to subagents ... the verbose output stays in the subagent's context while only a summary returns ... The subagent's own requests still draw on your usage" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- `/usage` on Pro/Max/Team/Enterprise shows "recent usage attributed to skills, subagents, plugins, and individual MCP servers, each shown as a percentage of the total". The `Prompt cache (main)` stats "cover the main conversation only, not subagents" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- Workflow fan-outs of same-prefix agents: Claude Code "holds all but the first for up to 5 seconds by default, so their first requests can read the prefix that the first agent cached" — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]
- Enterprise average is "around $13 per developer per active day and $150-250 per developer per month, with costs remaining below $30 per active day for 90% of users". This is not broken down by model or architecture — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- **Measured fan-out overhead:** "Two subagents cost 2.6x the sequential run in metered tokens. Five cost 3.2x." The title claims up to 5.9x, and the fan-outs "were never faster". Subagent system prompt measured "3,554 chars on Sonnet, 3,981 on Opus, and 4,509 on Fable" — [Systima: The Subagent Tax](https://systima.ai/blog/subagent-tax) [ANECDOTAL, via search snippet; page blocked, methodology unverified]
- Routing claims: pinning `model: opus` on a reasoner and `model: sonnet` on workers "shifts 80 to 90 percent of the tokens onto the cheap models" [ANECDOTAL]. "One Opus 4.7 orchestrator ... and four Sonnet 4.6 workers ... costs roughly 40% less than five Opus agents" [ANECDOTAL, OLDER MODEL]. "1 Opus 5 + 3 Sonnet 5 + 1 Haiku fleet costs $39.00 per day ... versus $81.25 for five Opus 5 agents — 52% less" [ANECDOTAL, OLDER MODEL] — [CloudZero](https://www.cloudzero.com/blog/claude-code-agents/); [Developers Digest](https://www.developersdigest.tech/blog/what-parallel-claude-agents-actually-cost); [MindStudio](https://www.mindstudio.ai/blog/smart-orchestrator-cheaper-sub-agent-models-claude-code) (search snippets; attribution of each figure to a specific page is uncertain)

### Inferences
- **The default setup gives no price saving.** A stock Opus 5.5 session delegating to Explore, Plan or general-purpose runs those sub-agents on Opus 5.5. To get the Sonnet or Haiku price you must do one of these:
  - set `model: sonnet` or `model: haiku` in the custom agent's frontmatter;
  - pass it per invocation;
  - set `CLAUDE_CODE_SUBAGENT_MODEL` plus `..._FORCE=1`, which also covers Explore and Plan.
- **Opus-5.5-era multi-agent vs single-agent gap.** Two things narrow the gap between Opus and Sonnet 5.5: Opus 5.5 is cheaper than Opus 5/4.x ($4/$20 vs $5/$25), and its cache reads cost the same as Sonnet's. So the ~40–52% fleet savings reported for older model pairs likely overstate the saving today. Those examples compare parallel fleets, which is not the same as a main thread plus occasional sub-agents.
- **When a sub-agent is net-positive (reasoning from primary mechanics).** It helps when the verbose output it absorbs, such as file dumps, logs or test output, would otherwise stay in the Opus main context and be re-read for many later turns. For example, 60K tokens of tool output kept in the main thread for 40 more requests costs 60K × 40 × $0.20/MTok = $0.48 in Opus cache reads, plus the 1h write of $0.48 (60K × $8). Handled by a Sonnet sub-agent it is roughly the sub-agent's own writes, reads and output, plus a small summary. Short, self-contained work, or work whose result the main thread needs anyway, is net-negative because of the 2.6–3.2x fan-out overhead reported above.

### Gaps
- I found no official Anthropic measurement for Claude Code specifically of main-context tokens saved vs total tokens spent when using sub-agents.
- I found no primary measurement of the Opus 5.5 orchestrator + Sonnet 5.5 sub-agent combination. All fleet cost figures found are for older models or are projections.

## 4. Subscription limits (Pro/Max): shared across models? separate buckets? does routing to Sonnet stretch limits?

### Takeaway
Official docs confirm a rolling **5-hour session limit** and a **weekly limit across all models**, and both are shared across models. Hitting either cannot be fixed by switching models. The docs also reference **model-family-specific limits** ("You've hit your Opus limit" / "Sonnet limit"), which *can* be escaped by switching families. Help-center pages say Opus consumes usage "several times more per turn" than Sonnet, so routing work to Sonnet or Haiku does stretch the shared limits. I found no official number for the multiplier.

### Cited Findings
- "'You've hit your session limit' or 'You've hit your weekly limit': a seat-based usage window on a subscription plan, **shared across all models**, so the developer can't restore access by switching models with `/model`." "After the model-specific 'You've hit your Opus limit' or 'You've hit your Sonnet limit' message, switching to a model outside that family with `/model` does keep the developer working" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- Teams/Enterprise: "each member's Claude Code usage draws from a per-seat allowance that resets on a rolling five-hour window and a weekly window. The allowance is shared with Claude chat and Cowork" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- Max: "Max 5x" is "five times the Pro plan's per-session usage allowance" at $100/month, and "Max 20x" is 20 times at $200/month. "Your session-based usage limit will reset every five hours"; "Max plans also have a weekly usage limit that applies across all models". The page does not describe a separate Sonnet or Opus weekly bucket — [Help Center: What is the Max plan](https://support.claude.com/en/articles/11049741-what-is-the-max-plan) [PRIMARY]
- Usage-limit best practices: the weekly limit is shown "for all models, and for Fable (if included in your plan)", which implies Fable has its own tracked limit. Factors that increase consumption include "Current conversation length", "Model choice", "Effort level", "Tool usage" and "Multi-step tasks". Cached content "counts less against your limits when reused" — [Help Center: Usage limit best practices](https://support.claude.com/en/articles/9797557-usage-limit-best-practices) [PRIMARY]
- "Opus costs several times more per turn than Sonnet, and Sonnet more than Haiku." The guidance is "Sonnet for most coding ... Opus when you're genuinely stuck ... Haiku for quick mechanical work" — [Help Center: Models, usage, and limits in Claude Code](https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code) [PRIMARY]
- Default model: "Pro, Max, Team, Enterprise, and Anthropic API: defaults to Opus 5.5" — [Claude Code model config](https://code.claude.com/docs/en/model-config) [PRIMARY]
- On Pro/Max, the subagent share of plan usage is visible in `/usage` attribution. It is "approximate and computed from local session history" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- Secondary claims: Max has "two weekly limits: one across all models and a second scoped to heavier models" (Opus). "Opus 5 consumes the weekly cap approximately 5x faster than Sonnet 5" — [usagebar](https://usagebar.com/blog/claude-weekly-limit-all-models-explained), [TrueFoundry](https://www.truefoundry.com/blog/claude-code-limits-explained), [ClaudeLog](https://claudelog.com/claude-code-limits/) via search snippets [SECONDARY, OLDER MODEL]. The 5x figure does not match the 2.5x API price ratio of Opus 5 ($5/$25) to Sonnet 5 ($2/$10) and is not stated by Anthropic.

### Inferences
- Whether routing to Sonnet stretches limits depends on how usage is metered. It very likely helps, since the official line is "Opus costs several times more per turn".
- Given part 2, routing **high or max effort** work to Sonnet 5.5 may **not** stretch limits: Sonnet uses about 60% more output tokens at max. That holds if plan metering tracks compute or token cost. Anthropic does not publish the formula, so this is unverified.
- **Cache TTL also affects limits.**
  - The main thread's 1h cache on a subscription means long idle gaps in the main thread don't cost a full rebuild.
  - Sub-agents' 5m cache means sub-agents that wait more than 5 minutes rebuild at write rates, and that counts against the shared limit.
  - Once a user is on usage credits, the main thread drops to a 5m TTL, which raises per-turn cost after short breaks.

### Gaps
- There is no official, current (5.5-era) statement of **how much** faster Opus 5.5 drains the 5-hour or weekly limits than Sonnet 5.5 or Haiku 4.5.
- I could not confirm whether Pro/Max currently have a separate Opus-only or Sonnet-only weekly bucket. The costs doc confirms that model-family limit messages exist, but not which plans have them or how big they are.
- I found no official statement on whether the effort level changes limit consumption beyond the tokens it adds. The best-practices page only lists "Effort level" as a factor.

## 5. Cost of long sessions: context growth, auto-compaction, /clear; is a fresh session cheaper?

### Takeaway
Claude Code re-sends the full context on every request, including every tool round-trip. Each request therefore costs about context size × cache-read rate, plus new tokens. Because the 5.5 models auto-compact only near **967K tokens** by default, a long Opus session can cost 10–30x more per request than a fresh one. `/clear` is free and resets this. `/compact` is itself a large but mostly cache-read request, and it is cheap only while the cache is warm.

### Cited Findings
- "Claude Code sends your full conversation with every request, and each time Claude uses tools it sends another request carrying that batch of tool results ... a one-line question in a session that has been open all day still draws usage for the whole conversation" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- "Compaction: `/compact` reads the conversation it summarizes, so compacting a large context is itself a large request. When you want a fresh start instead of continuity, `/clear` costs nothing" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- While the cache is warm, compaction "reads your prefix from the cache, so a mid-session `/compact` costs a fraction of what the context size suggests". After a break longer than the TTL, "the summarization request reprocesses the full history as uncached input", which makes `/compact` most expensive on resumed old sessions — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]
- Auto-compact: "Models running with a native 1M window, such as Sonnet 5, the Fable models, and Opus 4.7 and later on the Anthropic API, compact before the window fills, at about 967K tokens by default". It can be set from 100K to 1M with `/autocompact 500k`, `--autocompact` or `CLAUDE_CODE_AUTO_COMPACT_WINDOW`. "Sonnet 5.5 and Sonnet 5 always run with the 1M context window" — [Claude Code model config](https://code.claude.com/docs/en/model-config) [PRIMARY]
- The cache-lifetime rule: "your first message after a break longer than the cache lifetime misses the cache and reprocesses your full context. The lifetime is an hour on a subscription and drops to five minutes once you're drawing on usage credits; on an API key or cloud provider, it's five minutes by default." On Pro/Max, when resuming a large session after a long break, Claude Code "offers to resume from a summary" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- Other idle-time drains that re-send the full context: scheduled tasks and `/loop`, cross-session messages, goal check-ins (at most 3 idle per goal), subagents and workflows, and active teammates. Prompt suggestions are "mostly cache reads plus a few output tokens". Background summarization is "typically under $0.04 per session" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- "Unexpectedly high spend on an API or cloud-provider plan ... usually traces back to long sessions that were never cleared or to Opus left as the default model" — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]
- `/rewind` truncates back to an already-cached prefix, which is cheaper than compaction for abandoning a path. Editing CLAUDE.md mid-session doesn't invalidate the cache, and it doesn't take effect until `/clear`, `/compact` or a restart — [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching) [PRIMARY]
- Other recommendations: keep CLAUDE.md under 200 lines, move workflow instructions to on-demand skills, use hooks to filter verbose output, and keep MCP tools deferred (the default) — [Claude Code costs](https://code.claude.com/docs/en/costs) [PRIMARY]

### Inferences (illustrative arithmetic at list prices, not measured)
- **Per-request re-read cost on Opus 5.5** (cache read $0.20/MTok):

  | Context size | Cost per request |
  |---|---|
  | 30K | $0.006 |
  | 200K | $0.04 |
  | 500K | $0.10 |
  | 900K | $0.18 |

  A task needing 50 tool round-trips therefore costs about $0.30 in re-reads from a 30K start, vs about $5 at 500K and about $9 at 900K. That is before output.
- **Sonnet 5.5 re-read cost is identical ($0.20/MTok).** Moving a bloated long session to Sonnet does **not** reduce this part of the cost. Shrinking context with `/clear`, `/compact` or a lower `/autocompact` window does.
- **Cold-cache miss on a big Opus 5.5 context:**
  - 500K on the main thread with a subscription's 1h TTL costs 500K × $8 = $4.00.
  - The same miss at the 5m write rate costs $2.50.
  - On Sonnet 5.5 the equivalent miss costs half: $2.00 at the 1h rate, $1.25 at the 5m rate.
- **A fresh session (or `/clear`) is cheaper than continuing whenever the old context isn't needed.** The only switching cost is rebuilding a small prefix (system prompt, CLAUDE.md, git status), roughly tens of K tokens × $5–8/MTok ≈ a few cents. Sequential sessions in the same directory share the system-prompt cache when the git status snapshot matches.
- The 967K default auto-compact threshold means long sessions on 5.5 models can stay very expensive per turn for a long time before compaction kicks in. Setting `/autocompact` lower, for example 200–300K, is a direct cost lever.

### Gaps
- I found no official or independent measurement of dollars per turn vs session length on Opus 5.5 or Sonnet 5.5 in real Claude Code sessions.
- I found no data on the quality cost of compaction vs a fresh session, meaning rework caused by lost context.
