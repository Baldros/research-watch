# H1 — Harness Engineering — Living Research State

_Last reviewed: 2026-10-14 (first focused review)_

**Classification:** H1 denotes harnesses designed or externally optimized around fixed executor weights, including operational engineering, code-level audits and controlled configuration comparisons. Automatic, open-ended harness evolution with fixed weights belongs primarily to [H2](../h2-self-evolving/research.md); evolver-weight learning belongs to [H3](../h3-weight-harness/research.md). This is a **scheduler-owned** document. The parent [research.md](../research.md) is reserved for human cross-category synthesis and must never be changed by this watch. Consult the [taxonomy](../taxonomy.md).

## Current synthesis

1. **Harness design is a distinct optimization and measurement axis (supported, but workload-specific).** With a fixed executor, changes to tool interfaces, prompts and tool-boundary middleware can materially affect held-out task success. Do not attribute system gains to a better language model when the weights did not change; equally, do not assume an optimized harness transfers across models.
2. **Optimization must select a reliably good harness, not merely discover one high-scoring candidate (supported in controlled settings).** Selection variance, worst-condition lift, regression gates and search budget matter more than peak development score.
3. **The unit of evaluation is the (model, harness, workload, budget) configuration (supported by diagnostic evidence).** Different harness/model pairs exhibit distinct execution-alignment failures, latency, tool costs, and state divergence.
4. **Production harnesses are operated as versioned software with observability (industrial evidence).** Model-specific tool protocols, tool-error classification, A/B tests, offline evals, canaries and rollback are engineering practices. Industry claims lack independent reproducibility and should not be conflated with controlled scientific evidence.
5. **The foundational source-code anatomy by Barbaste et al. makes H1 concrete:** seven subsystems, 11 implementations, 29 patterns, and architecture-level contrasts such as Claude Code vs Codex. Its prescriptive guidance remains tentative because it is an observational audit, not a controlled performance comparison.

## Primary evidence (latest evidence available by 2026-10-14)

### A. Beyond Prompts: Measuring and Optimizing LLM Tool-Agent Harnesses — Airbnb (Cen Mia Zhao et al.)
- **Source:** https://arxiv.org/abs/2609.05736 (2026-09-04, v2 2026-09-09; research paper/preprint, arXiv metadata reports EMNLP 2026 acceptance).
- **Mechanism and weights:** constrained prompt/tool-boundary middleware edits with **fixed inner/executor model**. PRISM routes clusters of errors to prompt vs middleware changes; editable middleware comprises bounded intercept patterns, **not arbitrary rewriting of the agent loop**.
- **Evidence:** held-out average gains **+14.2 pp BFCL multi-round, +14.9 pp tau2-Retail, +10.1 pp tau2-Telecom**; introduces repeatability, worst-condition lift and selected-harness `RelLift95(B)` under fixed rollout budget. Component tests mainly credit repair-surface routing and edit-pattern constraints.
- **Interpretation:** defensible H1 measurement protocol, with an H2-adjacent automated optimizer inside a **restricted surface**. For taxonomy purposes treat the evaluation/engineering result as H1 and cross-link to H2, rather than asserting comprehensive self-rewriting runtime.
- **Limitations:** offline simulated tasks, small held-out scorecards and stochastic users; no multi-cycle production re-optimization. RelLift95 is an empirical diagnostic, not a formal future-deployment guarantee. Trial scoring and compute accounting need direct inspection.

### B. Harness-Bench: Measuring Harness Effects across Models in Realistic Agent Workflows — Yilun Yao et al.
- **Source:** https://arxiv.org/abs/2605.27922 (2026-05-27, arXiv preprint); code: https://github.com/Qihoo360/harness-bench .
- **Mechanism and weights:** vary harness *configuration* across model backends under common task environments, task oracles, resource envelopes and evaluation. No model or harness training in the benchmark.
- **Evidence:** the **paper's original version** reports **106 sandboxed tasks and 5,194 execution trajectories**, with systematic differences in completion, process quality and efficiency among pairings. A later project site describes a larger suite (300 tasks/20,454 runs); **do not mix this evolving website count with the versioned paper without matching the dataset version**.
- **Limitation:** representative configurations and offline fixtures, not isolation of the causal effect of any individual code component or proof of real deployment superiority. Important foundation for matched model–harness comparisons.

### C. Foundational H1 architecture reference: Harness Engineering — Anatomy, Architecture, and Evolution of Coding Agents

**Priority: foundational / anchor reading for H1.** This is the primary *source-level architectural map* for the study stream, **complementary to experimental harness evaluations** (such as PRISM and Harness-Bench). Refer to its component vocabulary when analyzing future H1 systems; do not treat its observed designs as established performance optima.

**Full title:** *Harness Engineering: Anatomy, Architecture, and Evolution of Coding Agents — A Source-Code Study of Eleven Systems*  
**Authors:** Paul Barbaste, Tristan Darrigol, Germain Vu, Tom Wiltberger.  
**Primary sources:** https://arxiv.org/abs/2609.00006 and https://arxiv.org/html/2609.00006 (arXiv preprint, submitted 2026-07-15, 83 pp.).  
**Study type:** qualitative comparative reverse engineering of **11 coding-agent harnesses**, plus **Omnigent** as an additional meta-harness contrast. The paper inspects roughly **4 million lines** of Python, TypeScript and Rust; defines **seven canonical subsystems**, **13 cross-cutting observations**, **29 design patterns** and **18 practitioner recommendations**. The same eight systems from an earlier edition were inspected again across approximately one quarter, offering longitudinal code-diff evidence. **The study does not benchmark or rank agents** (abstract; §§2–5, 13–16).

#### C.1 Scope, exact corpus and what counts as a harness

- **Corpus:** Claude Code, Codex CLI, Gemini CLI, Mistral Vibe, OpenHands, Aider, Mini-SWE-Agent, Hermes, Pi, OpenCode and OpenClaw. Omnigent is a **meta-harness** (coordinates vendor harnesses from above) rather than a twelfth implemented coding-agent loop.
- **Working definition:** agent = model + harness. A harness comprises the runtime coupling LLM inference to actions, tools, context, permissions, orchestration, recovery and extension interfaces. The executor/model and the software harness should remain distinct in evidence attribution.
- **Do not confuse:** a coding agent's harness with its SWE-bench *evaluation harness*; likewise, an imported framework library and a running agent harness are different software objects even as their boundaries converge.
- **Comparability warning:** the Claude Code analysis used a circulated March 2026 source snapshot, whereas Codex and much of the remaining corpus are pinned to July 2026 source trees. Product features evolve rapidly; conclusions about vendor internals are valid only for those snapshots.
- **Minimal vs maximal is not better vs worse:** Mini-SWE-Agent demonstrates a compact single-loop implementation (around 100 lines of core agent), whereas the large commercial CLIs implement extra reliability, portability, control and integration. File counts and self-reported task scores are **not controlled measures of quality**.

#### C.2 The seven-subsystem architecture — common checklist for future H1 analyses

| Subsystem | Design question | Implementation contrast identified in the study |
| --- | --- | --- |
| **D1. Agent loop** | How does inference → tool execution → observation proceed, including concurrency, limits and recovery? | Mini-SWE-Agent's linear loop; Codex's **Rust/Tokio async state machine**; Claude Code's streaming and safe tool batching; OpenHands event sourcing. |
| **D2. LLM integration** | How are prompt construction, provider dialects, streaming, caching, model selection and reasoning effort managed? | Claude Code's modular static/dynamic cached prompt; Codex's server-refreshed model instructions; provider-native vs multi-provider adapters. |
| **D3. Tools and actions** | Which tools are exposed, discovered, validated and executed? How are edits performed safely? | Deferred loading, native tool registries, structured file patches, concurrency gates, edit-repair strategies. |
| **D4. Memory and context** | How is limited context compacted and what survives between turns/sessions? | Linear transcript, threshold summarization, OpenHands pluggable condensation, Codex cross-session memory and repo-level Markdown instructions. |
| **D5. Safety and permissions** | Which actions need approval and which constraints are enforced by the runtime/OS? | Codex layered policy/hook/Guardian/OS sandbox; Claude Code classifier/hooks/approval and optional isolation; minimalist agents make different trust assumptions. |
| **D6. Orchestration** | Are there subagents, forks, mailboxes and shared state, or deliberately no delegation? | Claude Code recursive subagents; Codex thread trees; OpenHands branches; single-agent-only systems. |
| **D7. Extensibility** | How can users add capabilities and how can clients/agents drive the runtime? | SKILL.md bundles, MCP tools, ACP hosting, plugins, hooks, SDKs, and Pi's extension-first core. |

**Cross-cutting surfaces:** the human/program interface (CLI/TUI/IDE/server/SDK) and the session substrate (persistence, resume/fork, replay). These are important even when they cross several of D1–D7 (§2.3).

#### C.3 Concrete mechanisms: Claude Code, Codex and other designs

The comparison in the paper's Appendix **Table 18** is especially valuable for a grounded discussion of harness implementation:

- **Claude Code (source snapshot):** TypeScript streaming-event pipeline; dispatch batches only tools marked safe to run concurrently; modular prompt assembly with cached static prefix and dynamic suffix; deferred tool discovery; multi-layer permissions and recursively composed subagents.
- **Codex CLI (source snapshot):** Rust/Tokio state machine, streaming turn/session events and concurrency-aware tool-call runtime; per-model instructions from a refreshable catalog; structured tool schemas and deferred search; execution policies, hooks, approval reviewer and OS sandbox; parent/child thread orchestration with typed communications.
- **OpenHands:** event-sourced conversation engine, pluggable condensation and session branches; useful example of persistence as an architectural decision.
- **Mistral Vibe:** middleware pipeline separates limits, compaction and policy from the central agent loop.
- **Aider / Mini-SWE-Agent:** useful minimal or specialized counterexamples (edit-oriented verification; single bash action loop).

**Agent-loop clarification:** the loop above is the *software action cycle*, not the autoregressive token-decoding loop internal to a Transformer (§6). Independent subagents may repeat the same basic cycle in separate contexts; extra agent count is not evidence of greater cost-normalized capability.

#### C.4 Two empirical absences — precise interpretation

1. **No general-purpose agent-orchestration framework inside the studied runtimes.** The source audit looked for imports of LangChain, LangGraph, AutoGen, CrewAI and other common frameworks. All eleven core loops were built from host-language execution primitives (`asyncio`, JS async iterators/promises, Rust `Tokio`, and simple synchronous loops). **This is not “no third-party dependencies.”** Some systems import provider SDKs or transport helpers; OpenCode uses the Vercel AI SDK for LLM/tool plumbing, not a general-purpose agent orchestration framework (§13.2). It does **not** prove that frameworks are intrinsically worse; the paper's normative recommendation against them is stronger than the observational evidence.
2. **No embedding-based RAG for retrieval over the *repository code* in these eleven runtimes.** Instead they primarily use ripgrep, glob/grep, tree-sitter or symbol metadata, language tooling and repository instruction files. **This is not “no vectors anywhere”:** OpenClaw's conversation-memory subsystem uses hybrid vector + lexical retrieval, and Aider's optional documentation help is a distinct retrieval purpose. Code retrieval, tool discovery and persistent conversational memory must be classified separately (§9.7, §13.2).

This pair of absences suggests **testable engineering hypotheses**, not a universal ban on RAG or frameworks. A proper test would control the codebase, model, tools, token budget, cost, stale-index behavior and task mix across lexical/structural, vector and hybrid retrieval configurations.

#### C.5 Architectural evolution and the “platform turn”

- **Skills vs protocols:** observed SKILL.md support in **9/11**, MCP in **8/11**, and ACP in **6/11** systems. These are adoption counts for source snapshots, **not accuracy metrics**. Skills package procedural instructions, MCP connects tools and integrations, and ACP exposes hosting/interoperability.
- **Memory governance:** compaction, external persistent facts and repository-level instructions are different state surfaces; keeping provenance, deciding what to write, and handling contaminated memories are operational problems.
- **Policy as executable configuration:** the cross-release code-diff audit observed policy moving from plain prompt text toward typed/configured guards. This connects to the separate H1 evidence that side-effect authorization should be enforced outside model reasoning.
- **Diffusion of patterns:** the paper tracks shared hooks, plugin formats and session importers across products; claims about exact present-day vendor implementations must be re-verified after the sampled commits.
- **From tool to platform:** reusable harness SDKs, cross-vendor imports, plugins/skills, managed interfaces, marketplaces, and meta-harnesses support the authors' interpretation that harness products are becoming platforms. **This is an interpretive architecture/industry thesis**, not a quantitative effect-size estimate (§14).

#### C.6 Critical assessment and research uses

**Evidence strengths:** unusually broad **concrete source-code anatomy**, repeatable named subsystem vocabulary, comparisons among 11 diverse systems, specific implementation details, and a longitudinal view. It is an excellent **H1 foundational reference** for definitions, diagrams, implementation comparisons, and practitioner reading.

**Evidence limits:** nonrandom system sample; fast-moving snapshots; a special Claude Code source-access caveat; absent controlled head-to-head execution and cost benchmarks; no component-level causal ablation. The paper cannot show whether custom loops outperform LangGraph, or whether lexical retrieval outperforms semantic retrieval. It is also **not evidence of neural weight–harness co-training**. The paper's “model–agent co-design” means model-aware runtime integration, which remains H1 unless adaptation/training loops justify H3/H4.

**Study plan derived from the paper (manual research, not new scheduler scope):**
1. Use D1–D7 to classify the concrete architecture of each new H1 reference rather than treating “harness” as one indivisible component.
2. Compare **Codex vs Claude Code** in agent-loop concurrency, tool schema, prompt assembly, context/memory, sandbox/permissions and subagent coordination, always noting source versions.
3. Pair this architecture map with **controlled H1 studies** such as PRISM and Harness-Bench to determine whether observed design choices actually influence task outcomes at matched model and compute.
4. Reassess the “twin absences” as **falsifiable hypotheses**, tracking code retrieval separately from conversational memory and framework orchestration separately from provider SDKs.

**Priority classification:** **H1 / reference anchor / descriptive source-code evidence (high architectural value, limited causal value)**. Keep its paper details in the H1 living memory; do not overwrite the user's [cross-category research.md](../research.md) or add special automation/scheduler rules for it.

### D. Continually improving our agent harness — Stefan Heule & Jediah Katz, Cursor
- **Source:** https://cursor.com/blog/continually-improving-agent-harness (2026-04-30; production engineering report).
- **Mechanism:** per-model harness customization, tool-error taxonomy, model-specific interfaces, offline benchmark + live A/B instrumentation, keep-rate and anomaly alerts, ongoing context-selection redesign.
- **Reported evidence:** tool reliability raised to roughly **2–3 nines**, and unexpected tool-call errors reduced by roughly **one order of magnitude** in a focused sprint. An expensive summarizer offered negligible quality improvement and was not deployed; the report explicitly cautions against equating extra model spend with benefit.
- **Limitation:** internal metrics and experiments, not independently replicable; operational tool reliability is not the same as end-to-end task success.

### E. Supporting system-boundary evidence (retained from pre-split research)
- [Audit Without Verification](https://arxiv.org/abs/2609.07680) (2026-09): an LLM auditor can propagate upstream claims unless its evidence path is sufficiently independent. H1 relevance: verification architecture; **not** evidence that additional model roles alone help.
- [Beyond Agent Harnesses: Cross-Substrate Authority](https://arxiv.org/abs/2609.08472) (2026-09) and [Anthropic: How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude) (2026-05): deterministic authorization and sandbox constraints at execution boundaries; narrower controlled tests + production account, respectively.
- [EvoHarnessBench](https://arxiv.org/abs/2609.04280) (2026-09): expanding tools/skills may cause **harness-induced forgetting**; read the controlled benchmark details in H2's specialized synthesis before citing as a self-evolution algorithm.

## Contradictions / boundary conditions

| Question | Positive evidence | Negative or limiting evidence | Interpretation |
| --- | --- | --- | --- |
| Can harness changes help with weights fixed? | PRISM gains; Cursor operational metrics | PRISM small offline sets; retention regressions in EvoHarnessBench | Yes, with regression and cost controls. |
| Does an intrinsically stronger model remove harness sensitivity? | Some stronger models need fewer guardrails (Cursor) | Harness-Bench finds model–harness pairing effects; native tool contracts differ | Co-design for the actual model; no universal harness ranking. |
| Do extra prompts/agents improve reliability? | Directed prompt/middleware repairs help in PRISM | Auditor anchoring; unnecessary expensive summarization (Cursor) | Prefer failure-localized, testable changes and independent evidence paths. |
| Is an observed pattern a best practice? | Source-code audit finds common patterns | Observational sampling cannot identify causal performance effects | Use as hypotheses to test, not prescriptions. |

## Implementation/evaluation questions to track

- Per-model compatibility, **state and tool-call correctness**, rollback and model-switch semantics.
- Held-out changes after *multiple* consecutive harness updates; regressions on previously solved tasks.
- Task completion vs tool-call accuracy vs user satisfaction; cost per successful task, tokens, latency, tail reliability.
- Benchmarks that allow component-level causal isolation while preserving native agent-loop behavior.

## Review status and retained history

First H1-focused synthesis assembled on 2026-10-14 from existing parent research and primary sources. No claim of a **new October H1 breakthrough** is made merely because this specialized file was initialized. Historical multi-agent/AgentOps work continues to reside in the **untouched** parent document until the human researcher integrates it.
