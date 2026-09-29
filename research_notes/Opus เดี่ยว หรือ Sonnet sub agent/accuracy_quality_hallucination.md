# Accuracy, quality and hallucination: Opus 5.5 working alone vs an Opus orchestrator with Sonnet 5.5 / Haiku 4.5 sub-agents

Research date: 2026-09-29. Sonnet 5.5 was released 2026-09-28 (one day ago) and Opus 5.5 on 2026-09-22, so independent evidence on these two models is very thin. Most controlled single-vs-multi-agent evidence uses older or non-Claude models; each item below says which.

**How to read the verification status labels**
- **[VERIFIED-PRIMARY]**: I read this at the primary source, meaning anthropic.com or platform.claude.com pages fetched in this session. The fetch tool returns a summary made by a small model, not raw text, so exact wording may differ slightly.
- **[SECONDARY]**: I saw this only in search snippets, aggregators or blogs, not at the primary source.
- **Blocked sources.** The egress proxy blocked these hosts under organization policy: arxiv.org, www-cdn.anthropic.com (where the system card PDFs are hosted), artificialanalysis.ai, research.google, cognition.com, cruxevals.com, thenewstack.io. As a result, **I could not read the Opus 5.5 or Sonnet 5.5 system cards directly**, and every system-card and Artificial Analysis figure below is [SECONDARY].

---

## 1. Benchmarks and papers (2025–2026): single-agent vs multi-agent / orchestrator-worker, including equal-budget comparisons

### Takeaway
When budgets are matched, single agents match or beat multi-agent systems on reasoning and sequential tasks. Anthropic's own 2026 measurements show an orchestrator with cheaper workers is mainly a **cost/latency** tool that costs **roughly 10–12 accuracy points** relative to the frontier model working alone, even on corpus-scale work it is suited to. Multi-agent setups clearly win only when the work is parallelizable breadth or does not fit in one context, and those wins usually come from spending more tokens.

### Cited Findings

**Anthropic, "Optimizing for cost and intelligence" (platform.claude.com, 2026; Claude 5-generation models) [VERIFIED-PRIMARY]**
- **Corpus that exceeds one context window.** Setup: a 21.6M-token corpus of 14 Python packages with 130 planted defects. A Fable 5.1 coordinator over 25 Sonnet 5 workers cost 47–55% less than Fable 5.1 working solo at every effort level (about $240–260 vs $468–552 per task). It finished in about 2.3 h instead of 15–20 h. **Its accuracy was 10–12 points lower than Fable 5.1's best**, and "Fable 5.1 `high` effort still holds peak accuracy at ~2.2x coordinator cost." — [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- **Routine work (BrowseComp easy slice).** A Fable 5 coordinator with a Sonnet 5 worker cost about 50% less on average than Fable 5 alone, and its p90 cost was about one third ($12 vs $33). The most expensive solo run cost $84 and also got the answer wrong. The doc frames the orchestrator as "insurance against [the] cost tail." — [same](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- **Full BrowseComp, where the orchestrator lost.** Fable 5 alone at lower effort reached the same accuracy as the coordinator setup at 22–30% lower cost. Anthropic's statement: "An orchestrator buys something only when there is bulk to hand off… When the work is one dependent chain, or fits in a single context, the orchestrator pays for a plan, a handoff, and a merge that a single model gets for free." — [same](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- **Advisor pattern (the reverse direction: a cheap executor escalates to a stronger model).** On an internal agentic-coding benchmark, Opus 5.5 at `high` with a Fable 5.1 advisor scored 90.1% at $2.92 per attempt. That is +1.7 points over Opus 5.5 alone at `high` for about 2.1x the cost ("at edge of run-to-run noise"), and +3.5 points over Opus 5.5 at `medium` for about 3.5x the cost. The pairing gains most when the capability gap is large and the executor consults often: Haiku 4.5 + Opus 5 advisor showed a large gain, Opus 5.5 + Fable 5.1 a minimal one. **Warning from the doc:** at lower effort, the executor "can stop detecting it's stuck" and its consult rate collapses. On Chartography, Opus 5.5 at `low` consulted the advisor on 1 of 300 tasks and scored 61.7, **7 points below Opus 5.5 alone**. — [same](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- **Suitability table from the same doc.** The orchestrator suits "work >1 context window; many independent pieces; insurance against cost tail." It does not suit "one dependent chain; fits single context; task normally solved by single model at lower effort." — [same](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- **Anthropic's model-selection guidance.** "Most workloads start with Claude Opus 5.5." Haiku 4.5 is listed for "sub-agent tasks." Effort is often a better lever than switching models: "Tuning effort is often a better lever than switching models." The doc names two multi-model patterns: "an executor that escalates hard decisions to an advisor, and an orchestrator that delegates bulk work to lower-cost workers." — [Choosing the right model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model)

**Anthropic multi-agent research system (engineering blog, June 2025; Claude 4 generation) [VERIFIED-PRIMARY]**
- "Multi-agent system with Claude Opus 4 as the lead agent and Claude Sonnet 4 subagents outperformed single-agent Claude Opus 4 by 90.2% on our internal research eval." Token usage alone "explains 80% of the variance" on BrowseComp. Agents use about 4x the tokens of chat, and multi-agent systems about 15x. **This comparison was not at equal token budget.** — [Anthropic: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- The same post says multi-agent setups underperform where tasks "require all agents to share the same context or involve many dependencies between agents," for example "most coding tasks." — [same](https://www.anthropic.com/engineering/multi-agent-research-system)
- Some secondary blogs restate this result as "Opus 4.8 supervisor / Sonnet 4.6 subagents." That is a **misattribution**: the primary source says Opus 4 and Sonnet 4. — [thestackunderflow (secondary, via search snippet)](https://www.thestackunderflow.com/tutorials/subagent-isolation-context-rot/) vs [Anthropic primary](https://www.anthropic.com/engineering/multi-agent-research-system)

**Equal-budget academic comparisons (2026) [SECONDARY; arXiv blocked]**
- **Tran & Kiela, arXiv 2604.02460 (Stanford/Contextual AI, April 2026).** Models: Qwen3, DeepSeek-R1-Distill-Llama, Gemini 2.5; not Claude. When thinking-token budgets are held constant, single-agent systems "consistently match or outperform" multi-agent systems on multi-hop reasoning. The paper argues from the Data Processing Inequality that information passed between agents can only be lost. It notes that many prior benchmarks let multi-agent systems spend 2–4x more reasoning tokens. — [Beancount research log](https://beancount.io/bean-labs/research-logs/2026/05/31/single-agent-outperforms-multi-agent-equal-token-budget); [ResearchGate](https://www.researchgate.net/publication/403529711_Single-Agent_LLMs_Outperform_Multi-Agent_Systems_on_Multi-Hop_Reasoning_Under_Equal_Thinking_Token_Budgets); [DEV summary](https://dev.to/greza_dev/why-single-agents-beat-multi-agent-systems-at-equal-token-budgets-445c)
- **arXiv 2609.04217, "At Equal Inference Cost, Multi-Agent Structure Does Not Beat a Single Frozen Agent."** Setup: a frozen 7B backbone on ALFWorld, with a Planner-Executor-Critic team evolved through prompts. The full team scored 0.769 vs 0.754 for a single evolved executor (p = 0.80, not significant) while using 1.8x the evaluation calls. All the value came from the executor; the planner and critic prompts evolved to empty or low-impact text. Date note: the search snippet says "June 2026," but the ID 2609 implies September 2026. — [awesomepapers.io listing](https://awesomepapers.io/ai-agents/papers/2609.04217); [arXiv abs (not fetched)](https://arxiv.org/abs/2609.04217)
- Xu et al., "OneFlow" (January 2026), reportedly reached the same single-agent-wins conclusion across seven benchmarks and credited KV-cache reuse. I saw this only in a search summary and could not verify it. — [search summary via DEV/Beancount results](https://dev.to/greza_dev/why-single-agents-beat-multi-agent-systems-at-equal-token-budgets-445c)

**Google Research + MIT, "Towards a Science of Scaling Agent Systems" (arXiv 2512.08296, Dec 2025; not Claude-specific) [SECONDARY; research.google blocked]**
- The study compared five architectures (single-agent plus independent, centralized, decentralized and hybrid multi-agent) on Finance-Agent, BrowseComp-Plus, PlanCraft and Workbench.
- **Sequential planning (PlanCraft):** every multi-agent variant degraded performance by 39–70%.
- **Parallelizable financial analysis (Finance-Agent):** centralized coordination improved results by +80.9%.
- **Error amplification:** independent multi-agent systems amplify errors by up to 17.2x, while centralized designs contain errors best.
- The study's predictive model reached cross-validated R² = 0.513.
- — [InfoQ](https://www.infoq.com/news/2026/02/google-agent-scaling-principles/); [Towards Data Science](https://towardsdatascience.com/why-your-multi-agent-system-is-failing-escaping-the-17x-error-trap-of-the-bag-of-agents/); [HF paper page](https://huggingface.co/papers/2512.08296)

**Cognition [SECONDARY; cognition.com blocked]**
- **June 2025, "Don't Build Multi-Agents":** argued for a single thread plus a compression LLM, because parallel sub-agents make conflicting implicit decisions. — [Cognition](https://cognition.com/blog/dont-build-multi-agents); [Jason Liu summary](https://jxnl.co/writing/2025/09/11/why-cognition-does-not-use-multi-agent-systems/)
- **March 2026, "Devin can now Manage Devins":** reversed that position with a coordinator plus isolated sub-agent VMs, justified by "context accumulates, focus degrades, and the quality of each subtask suffers."
- **"Multi-Agents: What's Actually Working" (2026):** the pattern that works is "map-reduce-and-manage," and multi-agent systems work best "when writes stay single-threaded and the additional agents contribute intelligence rather than actions."
- — [Cognition blog post](https://cognition.com/blog/multi-agents-working); [FlowHunt summary](https://www.flowhunt.io/blog/multi-agent-ai-system/)

### Inferences
- Across Anthropic's own 2026 measurements and the independent equal-budget papers, the evidence is consistent: **at matched compute, the delegation pattern does not raise correctness.** It trades accuracy (about 10–12 points in Anthropic's corpus test) for cost and wall-clock time. The 90.2% gain from 2025 came from extra token spend and parallel breadth, not from delegation making answers more correct.
- For "Opus 5.5 alone vs Opus 5.5 + Sonnet 5.5/Haiku 4.5 workers," the most transferable evidence is Anthropic's Fable-coordinator + Sonnet 5 worker result: one tier down, 10–12 points lower accuracy, about half the cost. That tier gap resembles Opus 5.5 → Sonnet 5.5, which is narrow on many benchmarks. The gap to Haiku 4.5 is larger, so the accuracy loss would plausibly be larger too. This is an inference; no Opus 5.5 + Sonnet 5.5 orchestration result has been published.

### Gaps
- No published measurement of Opus 5.5 orchestrating Sonnet 5.5 or Haiku 4.5 workers, vs Opus 5.5 alone, on any task.
- I could not read the arXiv full texts (blocked) to extract per-benchmark numbers from 2604.02460 or check OneFlow.
- I found no equal-budget controlled comparison specifically for **document writing**.

---

## 2. Opus 5.5 vs Sonnet 5.5 on accuracy and hallucination

### Takeaway
On capability benchmarks Sonnet 5.5 is close to Opus 5.5: within about 0–8 points, and ahead on Terminal-Bench. Opus 5.5 leads on the hardest "would it be merged" coding (FrontierCode +8.2 points) and on factual knowledge. On AA-Omniscience, Sonnet 5.5 reportedly has a **lower hallucination rate (47% vs 59%)** but also lower accuracy (54% vs 66%), meaning Opus answers more and knows more, but guesses more when unsure. I could not verify the system-card figures, and the AA figures are secondary only.

### Cited Findings

**Anthropic launch pages [VERIFIED-PRIMARY]**

| Benchmark | Sonnet 5.5 | Opus 5.5 | Notes |
|---|---|---|---|
| Terminal-Bench 4.0 | 70.6% | 66.4% | Sonnet ahead |
| FrontierCode 1.1 (Main) | 46.2% (Max effort) | 54.4% | |
| CursorBench 4.0 | 55.5% | 57.8% | |
| GDPval-AA v2.1 | 1844 | 1846 | |
| AA-Briefcase v1.1 | 1811 | 1822 | |
| HLE (with tools) | 64.5% | 67.7% | |
| OSWorld 2.1 | 80.1% | 81.8% | |
| Chartography (no tools) | 61.6% | 64.4% | |

— [Introducing Claude Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5)

- The Sonnet 5.5 page states: "Opus 5.5 remains clearly stronger at complex, open-ended work requiring sustained judgment." The Sonnet page gives no hallucination or factuality metric. — [same](https://www.anthropic.com/claude-sonnet-5-5)
- Footnote on the Sonnet 5.5 page: "At Max effort, Sonnet 5.5 more often ran Claude Code's code-review skill, which splits the review across many subagents." — [same](https://www.anthropic.com/claude-sonnet-5-5)
- The Opus 5.5 page gives one factuality anecdote: on a financial research task, "16 out of 18 of Opus 5.5's reports cleared our quality bar, where any invented figure or quote would have failed." The page also features long unattended runs, with a tester reporting it "stayed on task for over 18 hours." — [Introducing Claude Opus 5.5](https://www.anthropic.com/claude-opus-5-5)
- Opus 5.5 vs predecessors: FrontierCode 54.4% (Fable 5.1 50.3%, Opus 5 48.0%); Terminal-Bench 4.0 66.4% (Fable 5.1 55.8%); AutomationBench 40.0%. — [Introducing Claude Opus 5.5](https://www.anthropic.com/claude-opus-5-5)

**Secondary**
- **SWE-bench Pro:** Opus 5.5 89.9% vs Sonnet 5.5 81.3%, with DeepSWE v1.1 at 74.2% for Opus 5.5. These are said to be "in Anthropic's System Card," but I saw them only in a search summary drawing on Vellum/llm-stats and could not check them against the system-card PDF (blocked). **Treat as unverified.** — [Vellum Opus 5.5](https://www.vellum.ai/blog/claude-opus-5-5-benchmarks-explained); [llm-stats](https://llm-stats.com/blog/research/claude-opus-5-5-launch)
- **AA-Omniscience (Artificial Analysis):**
  - Sonnet 5.5: 54% accuracy, 47% hallucination rate.
  - Opus 5.5: 66% accuracy, 59% hallucination rate.
  - Seen via a search-engine summary of AA pages; artificialanalysis.ai was blocked, so this is unverified. Note that AA's "hallucination rate" is the share of wrong answers among questions the model did not answer correctly, i.e. willingness to guess instead of abstaining.
  - — [AA Sonnet 5.5 article](https://artificialanalysis.ai/articles/claude-sonnet-5-5); [AA comparison page](https://artificialanalysis.ai/models/comparisons/claude-sonnet-5-5-medium-vs-claude-opus-5-5-high)
- **AA Intelligence Index:** Sonnet 5.5 (max) scores 56, "just 2 points behind Opus 5.5 (max), but at the highest Output Tokens per Task we've seen." — [Artificial Analysis on X](https://x.com/ArtificialAnlys/status/2104640155843989864)
- **Sonnet 5.5 behavioral audit:** on about 1,850 scenarios it "improves upon or matches Sonnet 5 across measures of alignment, resistance to misuse, and honesty." Seen in launch coverage; system card not read directly. — [Vellum / launch coverage via search](https://www.vellum.ai/blog/claude-sonnet-5-5-benchmarks-explained)
- **Prior generation (Opus 5 system card, July 2026), for context:** factual hallucinations were about 6% higher than Opus 4.8's, attributed to "an increased tendency to answer even when uncertain." — [Zvi on Opus 5 system card](https://thezvi.substack.com/p/claude-opus-5-the-system-card); [Opus 5 System Card PDF (not fetched)](https://www-cdn.anthropic.com/c5fbac3f0b1280a933ebd26d3cb8bb9f5bdeaf48/Claude%20Opus%205%20System%20Card.pdf)

### Inferences
- The AA-Omniscience pattern fits the Opus 5 system-card note that Opus models increasingly prefer answering to abstaining. **Opus is more often right but, when it doesn't know, is more likely to confabulate. Sonnet abstains more.** In an orchestrator setup, Sonnet 5.5 workers may therefore return "not found" more often and fabricated facts less often than Opus would. But they know less, so they miss more, and on knowledge-heavy research they would lose accuracy.
- The Sonnet/Opus gap is small on GDPval-AA (2 Elo) and Terminal-Bench (Sonnet ahead), which suggests delegating well-specified knowledge-work or terminal sub-tasks to Sonnet 5.5 costs little accuracy per sub-task. The gap is largest on FrontierCode (8.2 points), on the unverified SWE-bench Pro figure (8.6 points), and in open-ended judgment. Those are the tasks to keep on Opus.

### Gaps
- **I did not read either system card**, because www-cdn.anthropic.com was blocked. Figures I could not verify include SWE-bench Pro, the 100Q-Hard / SimpleQA-style factuality evals, and any "overclaiming task success" or reward-hacking metrics for Opus 5.5 and Sonnet 5.5.
- No Haiku 4.5 comparison was pulled for this generation.
- Independent evaluation of Sonnet 5.5 is roughly one day old. Beyond AA's index run, I found essentially no independent accuracy or hallucination studies.

---

## 3. Error propagation: summary loss, misreporting ("all tests pass"), hallucinated facts trusted by the orchestrator; verification loops

### Takeaway
Theory (the Data Processing Inequality), large failure taxonomies (MAST) and case studies agree that handoffs lose information and that sub-agents sometimes misreport. Verification is the weakest link. There is some evidence that an orchestrator told to double-check can catch fabricated results, but I found no rigorous rates.

### Cited Findings
- **MAST, "Why Do Multi-Agent LLM Systems Fail?"** (Cemri et al., arXiv 2503.13657, NeurIPS 2025; 2024–25 era models).
  - Dataset: 1,600+ annotated traces across 7 multi-agent frameworks; 14 failure modes; taxonomy built from 150 traces with κ = 0.88.
  - Failure categories: Specification issues 41.77%, **Inter-agent misalignment 36.94%**, **Task verification 21.30%**. "Disobey task specification" alone accounts for 11.8%.
  - [SECONDARY; arXiv blocked] — [alphaXiv](https://www.alphaxiv.org/abs/2503.13657); [Medium summary](https://thegrigorian.medium.com/why-do-multi-agent-llm-systems-fail-14dc34e0f3cb); [NeurIPS poster](https://neurips.cc/virtual/2025/poster/121528)
- **CRUX case study, "Can AI agents conduct open-ended AI research?"** (arXiv 2607.27191, July 2026): found "five instances of subagents hallucinating or misrepresenting results." In each case the orchestrator, "specifically instructed to double-check their work," uncovered the issue. [SECONDARY; cruxevals.com blocked] — [CRUX](https://cruxevals.com/crux/can-ai-agents-conduct-research/)
- **Practitioner issue reports (anecdotal) [SECONDARY]:**
  - A sub-agent reported "All existing tests pass" while grep showed zero tests touching the new symbols, so the claim was trivially true with no coverage.
  - A scout sub-agent read 672 KB of documentation and returned nothing, with no error flag.
  - The reporter describes "subagent fabrication (claim != diff)" as a recurring class, with a detection gap on the orchestrator side.
  - — [GitHub issue Osasuwu/jarvis #651](https://github.com/Osasuwu/jarvis/issues/651); [rickfelix/EHG_Engineer PR #8205](https://github.com/rickfelix/EHG_Engineer/pull/8205)
- **Trust hijacking:** "Trust the Brand, Lose Control" (arXiv 2609.32635, 2026) reports that changing a worker's claimed model name or tier changes whom the orchestrator trusts. That implies orchestrators weight sub-agent outputs by claimed identity rather than by checking them. [SECONDARY, snippet only] — [arXiv html](https://arxiv.org/html/2609.32635); [GitHub TrustFork](https://github.com/henrymao2004/agent-orchestration-safety)
- **Anthropic on its research system:** vague delegation causes sub-agents to "duplicate work, leave gaps, or fail to find necessary information." Early versions spawned "50 subagents for simple queries" and agents were "distracting each other with excessive updates." Manual testing was still needed to catch "hallucinated answers on unusual queries." [VERIFIED-PRIMARY; Claude 4 era] — [Anthropic multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- **Anthropic on handoff size:** each sub-agent "might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000–2,000 tokens)." That is a compression of roughly 10–50x by design. [VERIFIED-PRIMARY] — [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- **Error amplification:** up to 17.2x for independent multi-agent systems, with centralized orchestration containing errors best (Google/MIT, Dec 2025). [SECONDARY] — [Towards Data Science](https://towardsdatascience.com/why-your-multi-agent-system-is-failing-escaping-the-17x-error-trap-of-the-bag-of-agents/)
- **Clean-context reviewers:** Cognition's 2026 post reportedly counts reviewer-style agents that "contribute intelligence rather than actions" among the patterns that work. Anthropic's Sonnet 5.5 page notes Claude Code's code-review skill "splits the review across many subagents." Neither source, as I could access it, gave a bug-catch rate. [SECONDARY / VERIFIED-PRIMARY respectively] — [Cognition](https://cognition.com/blog/multi-agents-working); [Anthropic Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5)

### Inferences
- In an Opus-orchestrator + Sonnet/Haiku-worker setup, the dominant correctness risks are:
  - (a) Lossy 1–2k-token summaries that silently drop caveats or negative results.
  - (b) Unverified success claims such as "tests pass."
  - (c) The orchestrator trusting worker output without re-checking.
  
  Explicit verification instructions, or having the orchestrator re-run tests and grep itself, appear to reduce these risks, per the CRUX case study, but the evidence is anecdotal (n = 5 incidents).
- The Sonnet 5.5 abstain-more profile on AA-Omniscience (Section 2) could reduce fabricated facts from workers. Silent omission ("found nothing") then becomes the bigger worry.

### Gaps
- I found no quantitative study of the rate at which sub-agent summaries lose information or introduce hallucinations, or of the rate at which orchestrators accept false claims.
- I found no measured bug-catch rate for clean-context reviewer agents vs self-review in 2026 Claude models.

---

## 4. Does context isolation via sub-agents reduce context rot and improve accuracy?

### Takeaway
Context rot is real and well documented. Isolation plausibly helps, and Cognition's 2026 reversal and Anthropic's guidance are built on it. But controlled evidence that isolation **raises accuracy at equal compute** is lacking, and Anthropic's own 2026 corpus test still had the solo frontier model winning on accuracy.

### Cited Findings
- Anthropic defines context rot: "as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases." The degradation is "a performance gradient" rather than a cliff: "reduced precision for information retrieval and long-range reasoning." Anthropic recommends multi-agent designs for "complex research and analysis where parallel exploration pays dividends," compaction for "extensive back-and-forth," and note-taking for "iterative development with clear milestones." [VERIFIED-PRIMARY] — [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Chroma tested 18 frontier models; all degraded as input length grew even when the relevant information was present, with reported drops of 13.9–85%. [SECONDARY; Chroma study, 2025-era models] — [Morph summary](https://www.morphllm.com/context-rot)
- Cognition reversed its single-thread stance in March 2026 because "context accumulates, focus degrades, and the quality of each subtask suffers." [SECONDARY] — [FlowHunt](https://www.flowhunt.io/blog/multi-agent-ai-system/)
- **Counter-evidence:** on a 21.6M-token corpus that clearly exceeds one context window, Fable 5.1 working solo, using its own context management, still beat the coordinator + 25 Sonnet 5 workers by 10–12 points. [VERIFIED-PRIMARY] — [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- Tran & Kiela's result is framed "with perfect context utilization," which leaves room for multi-agent setups to help when context is degraded. [SECONDARY] — [Beancount](https://beancount.io/bean-labs/research-logs/2026/05/31/single-agent-outperforms-multi-agent-equal-token-budget)
- Context-management tools have costs of their own. On a 20-issue triage run, context editing cost 74% more and saved nothing; on a run with 2.6x the tokens it saved 32%. [VERIFIED-PRIMARY] — [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence)
- Opus 5.5 is marketed for "long and sprawling jobs like codebase-wide migrations and audits," with an 18-hour unattended run cited. That suggests the current Opus handles long single-agent sessions (with compaction) better than older models did. [VERIFIED-PRIMARY, vendor claim] — [Introducing Claude Opus 5.5](https://www.anthropic.com/claude-opus-5-5)
- A 2026 study reports that "context rot persists at the 2026 reasoning frontier under cross-domain noise injection," and that more deliberation does not cure it. [SECONDARY, snippet only; paper unidentified] — [search result set incl. TDS on Claude Code context rot](https://towardsdatascience.com/governed-context-managing-context-rot-in-claude-code/)

### Inferences
- Isolation mainly protects the **main thread's** context budget and wall-clock time. On current-generation models, the accuracy lost to lossy handoffs and weaker workers appears to outweigh the accuracy gained from a cleaner lead context, at least in the one controlled Anthropic test available. Isolation most plausibly improves accuracy in read-heavy exploration that would otherwise flood the lead's context with noise.

### Gaps
- I found no controlled 2026 study that holds compute equal and compares Opus-class solo+compaction against an orchestrator+isolated workers specifically to measure context-rot effects.

---

## 5. Which task types benefit and which suffer

### Takeaway
Delegation helps **parallelizable, read-only breadth**: multi-source research, corpus scanning, multi-perspective review. It hurts **sequential, tightly coupled work**: most coding and long coherent documents. For writes and final synthesis, a single strong agent (Opus 5.5) should own the thread.

### Cited Findings
- **Benefits:**
  - Breadth-first research ("heavy parallelization, information that exceeds single context windows, … numerous complex tools"). — [Anthropic multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) [VERIFIED-PRIMARY]
  - Parallel financial analysis: +80.9% with centralized coordination. — [InfoQ on Google/MIT](https://www.infoq.com/news/2026/02/google-agent-scaling-principles/) [SECONDARY]
  - Routine, many-independent-pieces work, mainly on cost. — [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence) [VERIFIED-PRIMARY]
- **Suffers:**
  - "Most coding tasks" and anything needing shared context. — [Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system) [VERIFIED-PRIMARY]
  - "One dependent chain" or work that "fits in a single context." — [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence) [VERIFIED-PRIMARY]
  - Sequential planning: −39% to −70%. — [InfoQ](https://www.infoq.com/news/2026/02/google-agent-scaling-principles/) [SECONDARY]
  - Multi-hop reasoning at equal budget. — [Beancount/Tran & Kiela](https://beancount.io/bean-labs/research-logs/2026/05/31/single-agent-outperforms-multi-agent-equal-token-budget) [SECONDARY]
- **Coding compromise:** keep writes single-threaded and use extra agents for intelligence such as review and exploration, not actions. — [Cognition, via FlowHunt](https://www.flowhunt.io/blog/multi-agent-ai-system/) [SECONDARY]
- **Review:** Anthropic's code-review skill splits review across many sub-agents. — [Anthropic Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5) [VERIFIED-PRIMARY]
- **Output format has little effect on correctness:** a triage agent's one-line vs memo output formats were "within run-to-run noise" on correctness (78–85%) but differed about 3x in cost. — [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence) [VERIFIED-PRIMARY]

### Inferences
- **Document writing.** No direct evidence was found. By the same logic (a coherent voice, cross-section consistency, one dependent chain), a single Opus 5.5 author is likely better for final drafting. Sub-agents are best limited to gathering sources or fact-checking sections in parallel, with the orchestrator verifying cited facts before they enter the draft.
- **Research/reading.** An Opus lead with Sonnet 5.5 readers is well-suited. Sonnet 5.5 is close to Opus on GDPval/Briefcase, and reportedly abstains more on unknown facts. The orchestrator should still re-verify key numbers against sources, because the failure is typically silent omission or unverified claims rather than obvious error.
- **Agentic coding.** Prefer Opus 5.5 alone, or Opus 5.5 with read-only exploration/review sub-agents. Delegating implementation to Sonnet 5.5 costs about 8 points on FrontierCode-style mergeability. Delegating to Haiku 4.5 likely costs more; no current-generation number was found.

### Gaps
- No 2026 benchmark directly measures multi-agent vs single-agent **document-writing** quality or factual accuracy.
- No current-generation Haiku 4.5-as-worker accuracy data was found.
