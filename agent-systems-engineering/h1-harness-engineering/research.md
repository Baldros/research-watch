# H1 — Harness Engineering — Living Research State

_Last reviewed: 2026-11-09 (security, budget confounds, model–harness fit, verification, and runtime contracts)_

**Classification:** H1 denotes harnesses designed or externally optimized around fixed executor weights, including operational engineering, code-level audits and controlled configuration comparisons. Automatic, open-ended harness evolution with fixed weights belongs primarily to [H2](../h2-self-evolving/research.md); evolver-weight learning belongs to [H3](../h3-weight-harness/research.md). This is a **scheduler-owned** document. The parent [research.md](../research.md) is reserved for human cross-category synthesis and must never be changed by this watch. Consult the [taxonomy](../taxonomy.md).

## Current synthesis

1. **Harness optimization is real but the ranking is conditional on model, harness, tasks and budget.** PRISM's constrained prompt/tool edits improve held-out success with a frozen model; MedicalHarness finds harness and model–harness interactions. The 66-configuration *Finding the Right Fit* study finds actual **model ranking reversals** after switching harnesses, sometimes alongside a lower cost. No vendor harness or model is universally optimal.
2. **Reliable candidate selection matters more than a single high development score.** Track repeatability, worst-condition lift, held-out tasks and candidate-search budget; prompt/middleware tuning and code rewrites are distinct action spaces. Retention can regress after seemingly beneficial changes.
3. **The 11-system source-code anatomy remains the foundational descriptive H1 map.** Seven subsystems and 29 patterns document concrete runtime designs; it does not establish which design wins in controlled tests or prove vector search/frameworks cannot help.
4. **Harness source correctness is separately measurable from downstream agent success.** HarnessTester shows contract-guided tests discover more actual agent-runtime defects than generic test generation. Doc2Harness uses mock-model scripted execution and positive-control gates to test whether a constructed full harness implements expected permissions, state, tool and interface behavior. Neither benchmark proves deployed agent-task lift.
5. **Permissions produce a security–utility trade-off, not a scalar 'safer is better' ordering.** HarnessSecurity-Bench fixes the main executor and toggles native controls: auto-approval raised benign completion by **18.3 pp** and attack success by **66.4 pp** in the study's tasks. Restrictions and command filters differ dramatically in utility cost; alternate allowed paths can bypass a rule.
6. **Budget limits are causal system variables, not harmless evaluation details.** ToolBottleneckBench finds an apparent large-tool-menu deficit mostly caused by completion-cap truncation in the heavy-truncation stratum. Record actual completion/episode/call limits, tool-menu exposure, truncations and total cost before diagnosing model inability.
7. **Verification needs an evidence path independent of the claim being checked.** Audit Without Verification warns about relayed conclusions. VeriHarness improves selection and revision with tool-backed disagreement and consensus checking over a **fixed pool of 10 rollouts**; these gains come with separate verification compute and some grader-based endpoints. More model votes alone are not independent evidence.
8. **Coordination is useful when it diagnoses and repairs failures, not simply because more agents participate.** Maestro's Lean coordinator can retract flawed proof plans and significantly outperforms decomposition-only in its particular theorem-proving setting, while adaptive multiworker compute and task specificity limit its generality.
9. **Production operation is versioned engineering, not a one-time prompt.** Cursor's published tool-reliability improvements support telemetry, canaries and rollback but remain proprietary operational evidence; no controlled prospective replication establishes a universal production recipe.
10. **Auxiliary decision controllers are workload-dependent.** JevAgentHarness trains a small auxiliary policy while keeping the main task executor frozen. Large savings in discrete-action environments (81–89%) shrink to **7–10.5%** in its coding examples; auxiliary training and calibration must be amortized, and this is not H3 editor training.
11. **Report evidence type explicitly.** Source-code audits (descriptive), scripted mock-executor contract tests (harness code behavior), controlled fixed-model runtime interventions (causal within workload), and production logs (ecological but not independently reproduced) support different conclusions. Cross-paper headline scores are not directly comparable.

## Primary evidence (reviewed through 2026-11-09)

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

### F. Complex Agents, Shallow Tests: Demystifying and Enhancing Test Adequacy of Agent Harness in the Wild — Yifan Xiong et al.

- **Primary source:** https://arxiv.org/abs/2610.04921 (2026-10-04, arXiv preprint); **focus:** harness-code testing rather than model training or agent benchmark optimization.
- **Problem/mechanism:** harness logic processing LLM-derived values (**LLM-dependent harness**, LDH) is poorly exercised by conventional tests. The authors study **10 open-source agent systems** and construct **HarnessTester**, an LLM-based test generator that recovers *cross-component contracts* (valid tool payloads, context objects and expected control flow) to build realistic fixtures and target LDH branches. Contract-aware source retrieval helps reach paths missed by generic tests.
- **Controlled comparisons:** against Python/TypeScript general-purpose generators with the same underlying GPT-5-mini model and per-project time budget, HarnessTester reports **75.95% higher line-coverage gain, 84.76% higher branch-coverage gain, and 69.89% higher mutation-score gain** than the strongest baseline selected per project/metric. These are **relative improvements in coverage gains**, not coverage rates or end-to-end agent accuracy gains.
- **Bug evidence:** on a historical set of **250 harness defects**, it rediscovers **43** versus **2** for the strongest baseline (at a higher total generation cost, but lower reported cost per revealed historical bug). In a separate test on recent revisions, the authors report **122 real bugs = 88 previously unknown + 34 known**; **69 of the 88 new bugs** had been developer-confirmed, and 64 fixed or assigned an open fix by the paper's reporting date. These counts refer to a particular audit and should not be generalized into production defect rates.
- **Ablation:** removing contract awareness drops historical bugs revealed from **43 to 15**. This isolates a mechanism: **test validity depends on reproducing the real contract between model output, harness input and downstream action**.
- **Limits:** selected open-source Python/TypeScript systems; structural coverage and detected bugs are not sufficient to establish deployed end-to-end reliability; generated assertions/mocks can be invalid; confirmation depends on maintainers and available bug reports; code and model generations incur cost. Strong **H1 verification/testing** evidence, not H2 autonomous self-evolution.

**H1 implication:** add LDH-specific integration and differential tests, exercise malformed/partial tool outputs, asynchronous control-flow transitions, memory/context changes and recovery paths, and track code-coverage *and* contract-correctness separately from agent task outcomes. Complements the Barbaste et al. architecture audit with controlled testing evidence.

### G. MedicalHarness: A Controlled Evaluation of LLMs and Agent Harnesses on Medical Tasks — Ziqing Wang et al.

- **Primary source:** https://arxiv.org/abs/2610.05778 (2026-10-05, arXiv preprint); repository: https://github.com/REAL-Lab-NU/MedicalHarness .
- **Protocol:** **107 medical-oriented tasks** spanning tool use, long-context recall, clinical-trial planning and web workflows; five **frozen open-weight models** × five existing harness implementations (**OpenClaw, Hermes, ZeroClaw, Codex, Claude Code**) × **three seeds = 8,025 evaluation episodes**. The team serves one checkpoint per model through a common interface to better isolate harness effects. These are experimental task environments, not clinical validation.
- **Variance decomposition:** averaged over four task environments, **model ≈57.5%**, **harness ≈12.4%**, **model–harness interaction ≈11.9%** and **run-to-run ≈18.2%** of measured outcome variance (paper also rounds these to 58/12/12/18). These are **ANOVA-style variance contributions in this experimental design**, not general laws about all agents. Effects differ markedly across environments; no universally best harness emerges.
- **Mechanistic intervention:** MH-Lab changes **context management**, **planning** or **direct tool exposure** *one at a time*, holding model, task instructions, underlying tools, budget and loop constant. Some removals substantially worsen a particular model-task pairing while being neutral or occasionally beneficial elsewhere, supporting **conditional, model-aware harness engineering** over blanket features.
- **Operational diagnostics:** task scores combine answer production and task correctness. Under the native context windows, compaction happens in only **2–5% of episodes**, limiting how much variance it can explain there. Failure modes include submitting no answer, the wrong value after selecting a correct calculator, overproducing trial candidates and partial web-workflow completion.
- **Costs/limitations:** significant cross-harness differences in token use and tool use complicate cost-normalized ranking. The task suite is narrowly medical (107 tasks), with only three seeds and selected model/harness versions; vendor-specific adapters/configuration changes may affect comparability. Paper statistics do **not** imply the harness dominates the model, and some domain-level leader gaps have uncertain intervals. Medical-task success should never be interpreted as clinical safety.
- **Classification:** unusually informative **H1 controlled evidence**, complementing the architecture-only Barbaste et al. paper and Harness-Bench. Executor weights are fixed; no learned evolver (H3) or executor co-adaptation (H4).

**H1 implication:** compare **(model, harness, task)** triples and component toggles, separate answer-delivery reliability from semantic correctness, and track per-task success, latency, tokens and failure reasons. The paper provides a concrete protocol for testing architectural hypotheses from the eleven-harness source audit.

### H. JevAgentHarness: Benchmarking, Deploying and Training Decision Models That Share Control with LLMs — Kunlun Zhu et al.

- **Primary source:** https://arxiv.org/abs/2610.21412 (2026-10-15, arXiv preprint); implementation/project: https://github.com/JevAgentHarness/JevAgentHarness .
- **Harness mechanism:** intercept recurring **discrete decisions** (tool choice, argument from a fixed set, reflection/stop/routing) and ask a cheap small decision model. The harness accepts it only above a **per-decision confidence threshold** fitted from *shadow-mode agent trajectories*; otherwise the larger LLM decides. Modes include cascade, replace, assist, guard and trigger. The large **executor LLM remains frozen** in the reported routing evaluations.
- **Important classification nuance:** the **auxiliary 4B decision model is itself trained** (SFT on recorded visited states, including post-error states, then GRPO with task outcome and LLM-call cost). This is a **learned controller inside an H1-style engineered harness**: not “all weights frozen”, not an H3 learned *harness-editing evolver*, and not H4 training the main task executor jointly with a changing harness. Note this explicitly when comparing trained vs off-the-shelf routing.
- **Results:** in targeted customer-service/household agent settings with the specially trained 4B controller, the authors report **81.1–89.0% lower inference cost** and up to **2.5×** speedup within a prespecified success non-inferiority margin (mostly 3 percentage points). On shifted DecisionGym tasks, an agent controlled by trained AgentDecision-4B solves **91.5%**, versus **82.0%** with Jev, under the reported protocol. These are **not generic coding-agent savings**.
- **Counterevidence:** in four coding-agent comparisons, small-model routing saves only **7.0–10.5%** of cost; performance can improve or regress depending on harness/model. Sending **every** tool decision to Jev without calibrated deferral reduces success by **19.0 points** in one setting. On tau2-bench, one chosen delegation fails its transfer/non-inferiority test. Reported 89% savings are for suitable action-selection workloads with trained auxiliary models, not arbitrary long coding agents.
- **Costs and generalization:** threshold fitting needs held-out agent traces and may need retraining when tasks/tools/model APIs change. The 4B controller introduces GPU serving and training cost; paper estimates one training+calibration process at **$192.93** and break-even after **5,759 episodes** in a specific decision-gym configuration. Cost-normalized comparisons must incorporate serving, training amortization and confidence-calibration drift. Predominantly English/synthetic task settings; preprint without independent production replication.
- **Classification:** **H1 (model-aware runtime delegation) / adjacent learned auxiliary policy**. Related to the taxonomy's model–harness boundary, but outside the strict H2→H4 adaptation mechanisms unless the *harness itself* is evolving or the **main executor** is retrained.

**H1 implication:** evaluating decision routing solely by isolated next-tool accuracy is misleading: use **trajectory-level and task-level closed-loop success**, fitted deferral thresholds, error-recovery states and matched price/latency metrics. A smaller controller may be cost-effective when decisions are discrete and confidence calibration is reliable; the same claim is much weaker for free-text-heavy coding tasks.

### I. HarnessSecurity-Bench: Do Security Mechanisms Really Protect Coding Agent Harnesses? — Zhengyang Zhu et al.

- **Primary:** https://arxiv.org/abs/2610.07639 and https://arxiv.org/html/2610.07639 (submitted **2026-10-06**; arXiv preprint, *not* a new November release).
- **Design:** an audit of **40 harnesses × 10 mechanisms = 400** source/documentation cells; separately, a **23-task**, **six-harness** sandbox experiment evaluating **nine mechanisms** across five adversarial surfaces, **2,500 trials**, **81,155 tool calls** and >2.2B tokens. Keep documentary coverage distinct from systems actually tested. ON/OFF policy comparisons hold **GLM-5.2 main executor fixed**; deterministic benign and attack oracles are separate. When auto-approval is OFF, a **Qwen3.8-27B LLM reviewer**, *not a human*, handles permission prompts.
- **Mechanism-specific results:** auto-approval **OFF→ON:** legitimate utility **77.1→95.4% (+18.3 pp)**, attack success **29.2→95.6% (+66.4 pp)**. Network isolation ON: attack success **57.1→0.8%** with **−24.5 pp benign utility**; read-only ON: **8.3→0.8%** attack success with **−34.0 pp utility**. Command allowlisting reduces attack success by **48.1 pp** with **−1.5 pp utility**; denylisting reduces it **40.4 pp** with **+5.2 pp utility**, *within selected test mixtures*. Alternative routes through permitted commands can remain reachable.
- **Conclusions/limitations:** stronger evidence for **enforced runtime permission boundaries** and paired benign/adversarial evaluation than prompt-level security. Attack-success proportions are *not* field probabilities; scripted tasks, one fixed main model, native-control heterogeneity, a synthetic approval reviewer and unresolved closed-source audit cells limit generalization. Some controls are not comparable with independent user approval. Report risk exposure and legitimate workflow success jointly, not just safety score.

### J. ToolBottleneckBench: Measuring Tool-Agent Performance under Controlled Pressures and Fixed Harness Budgets — Zichuan Xu et al.

- **Primary/code:** https://arxiv.org/abs/2610.21098 ; https://arxiv.org/html/2610.21098 ; https://github.com/zichuanxu/tool-bottleneck-bench (submitted **2026-10-15**, revised Oct 16; preprint).
- **Protocol:** **21 fixed model–harness configurations** across compiler-generated enterprise tool tasks with paired interventions on catalog, visible menu, workflow and returned-state encoding. An exact nine-obligation grader records correctness and termination. Base limits include **1,024 per-response completion tokens**, an episode budget and call cap; no neural weights updated.
- **Budget mechanism:** increasing visible menu **8→128** schemas costs **−12.3 pp** verified task success pooled over sampled conditions (95% clustered CI [−14.6, −9.9]); of the **six truncation-heavy configurations**, low-cap menu penalty is **−43.7 pp** [−48.5, −39.1] and falls to **−2.7 pp** [−4.7, −0.9] at **16,384 completion tokens**. The +41.0 pp interaction is **not** the mean of all 21 configs; a small residual penalty remains. In the main dataset **91.5% of failed episodes end at a harness limit** (do not conflate this with a model reasoning error).
- **Diagnostic contradiction:** first-failed-obligation + termination flag predicts which pressure caused failure with only **48.9% balanced accuracy** in a 127-packet analysis, *worse than simple tool-call counts (54.8%)*. Targeted connector/router fixes can improve outcomes compared with rerunning a different model, but the methods may exploit controlled public workflow data.
- **Limitations:** simulated deterministic workflows, uneven effective reasoning-token accounting, finite repeats, some oracle-informed tool routing and no production deployment. For failure triage, log the **effective response cap, first tool-call attempt, truncation, episode budget and terminal reason**, and compare repair against same-budget model/config switches.

### K. Finding the Right Fit: Model–Harness Interactions across Agent Tasks — Yixuan Li et al.

- **Primary/code:** https://arxiv.org/abs/2610.00917 ; https://arxiv.org/html/2610.00917 ; https://github.com/liyix/finding-the-right-fit (submitted **2026-10-01**, preprint).
- **Protocol:** **66** configurations: 4 configurable harnesses (OpenHands, DeepSeek Harness, PI, openJiuwen) × 5 fixed model checkpoints × 3 task collections, plus native Claude Code and Codex pairings; **6,204 scored trajectories** released. Task sets include TUA-Bench, ALE-CLI and 63 selected Terminal-Bench 4 tasks. Models are held fixed *within* same-model harness comparisons; **harness-internal tools, settings and budgets are not uniformly matched**.
- **Rank reversals:** on Terminal-Bench 4, Claude vs GPT scores **36/63 vs 31/63** under OpenHands but **19/63 vs 38/63** under PI — the same model ranking reverses with the runtime. Native pairing is not uniformly best: PI–GPT scores **60.32%** and **$4.66/task**, while DeepSeek Harness–GPT scores **52.38%** and **$19.94/task** under paper prices. For **4/5 models**, the best available harness changes across benchmarks; openJiuwen is a stable best for Kimi among examined options.
- **Diagnostic insight:** failure repair is mostly **model-initiated** (180/192 coded events) and depends on whether the harness makes failures actionable. The authors infer that lean PI suits GPT while openJiuwen suits error-prone tool-call formats from Kimi.
- **Limitations:** one counted run per task, selected benchmarks, uncontrolled budget/tool/runtime differences and observational case analysis; this establishes **configuration interactions and cost differences**, not causal superiority of an isolated component. Cost depends on vendor pricing/cache semantics.

### L. Doc2Harness: Benchmarking LLMs on Long-Horizon Harness Engineering of Coding Agents — Yuyao Ge et al.

- **Primary:** https://arxiv.org/abs/2610.20344 ; https://arxiv.org/html/2610.20344 (**2026-10-14**, preprint).
- **Scope and separation:** a builder model, operating inside a fixed builder harness, receives an interface skeleton and behavioral contract synthesized from **12** agent harnesses, then creates a complete candidate. A **scripted mock executor** runs that *frozen source artifact* in independent test containers. **328 deterministic cases** across permissions, file operations, session state, direct/headless/interactive interfaces; observable channels are model requests, filesystem, exit code and command output. **Positive controls** reject vacuous passes (e.g. a tool that rejects all reads cannot pass 'blocks forbidden read').
- **Findings:** **19** builder models × **3 independent candidate builds**, top **53.05% ±0.25** across 328 cases, bottom **27.44%**. Hard failures cluster on cross-module behavior and boundary conditions. These are **test pass rates for code the builder wrote**, *not task-solving rates of a deployed agent* and not proof that a stronger executor causes better harness outcomes.
- **Limits:** contractual coverage is partial and benchmark-authored, no natural continuously deployed agent, builder-model differences remain, no conventional unseen-domain transfer or prospective security proof. Strong complementary H1 *artifact-level* benchmark vs HarnessTester’s targeted historical bug discovery.

### M. Maestro: Agentic Formal Proof Development by Coordination — Haoyu Li et al.

- **Primary:** https://arxiv.org/abs/2610.25920 ; https://arxiv.org/html/2610.25920 (**2026-10-19**, preprint).
- **H1 mechanism:** with each **fixed frontier backbone**, a global Lean-proof coordinator dispatches specialists and can retract mistaken upstream decompositions on failure reports, versus native Claude Code/Codex. Kernel verification supplies independent correctness evidence.
- **Results/ablation:** on June 2026 arXivLean, **Claude Opus 5: 34/48** proved by Maestro vs **19/48** by the native agent. A decomposition-only baseline lacks error diagnosis/global replanning and contributes little or no improvement on FATE-X: with Claude Fable 5 **76 vs 77** solved (decompose vs plain), Maestro **91**; with GPT-5.6 Sol **72 vs 70**, Maestro **84**. Thus benefit is not simply 'more subagents'.
- **Budget:** high variability and adaptive inference. Authors compare some solved-case token medians to three plain-agent attempts, but these **conditional medians do not normalize all attempts, wall clock and full price**; difficult successful cases can consume hundreds of thousands or millions of tokens. Also cross-system leaderboard comparisons use different models/budgets and are *not causal*.
- **Limits:** formal mathematics/Lean niche, single-run main results and selected rerun robustness, specialized verification interface; transfer to loosely specified software work or typical production agent teams unproven.

### N. VeriHarness: Scaling Agentic Verification for Long-Horizon Tasks — Caiqi Zhang et al.

- **Primary:** https://arxiv.org/abs/2610.00972 ; https://arxiv.org/html/2610.00972 (submitted **2026-10-01**, preprint).
- **Design:** start with a **fixed pool of N=10 generator rollouts** per task; reuse the frozen generator model as a tool-capable verifier without final-answer gold rubrics at test time. **Disagreement resolver** checks competing claims with environmental evidence; **consensus challenger** tests shared assumptions; adjudication selects and can revise a workspace artifact. Fixed pool isolates verification selection (not full generation budget). Tested across 5 long-horizon workspace benchmarks, 2 models and 3 verifier seeds.
- **Scores:** Gemini 3.5 Flash single-rollout mean **47.2**, select **51.6**, select+revise **53.4**; Claude Opus 4.8 single **49.6**, select **53.7**, revised **56.1**. Values average **five benchmark-specific score scales**, not universally a gain in pass probability. Majority vote and naive best-of-N trail evidence-grounded checks. They also explore external verification-skill revisions, which **do not update neural weights**; distinguish this H1 verifier design from H2 memory self-evolution.
- **Costs:** verification-only selection estimate/metering **$1.44/task (Flash), $3.92/task (Opus)** with cache rates; revision roughly **3×** selection. The 26k-rollout released pool cost **>$100k** and is **not included in those incremental selection numbers**. Some baseline costs are estimated; CLI implementations can cost **2–3×** the authors’ streamlined verifier runtime. Cached token pricing materially affects comparisons.
- **Limitations:** workspace-specific scoring/LLM rubrics, selected test distributions, consensus can preserve shared errors, no production/shifted-distribution test. Verify whether both independent evidence and final artifact quality improve at **matched total inference**.

## Contradictions / boundary conditions

| Question | Positive evidence | Negative or limiting evidence | Interpretation |
| --- | --- | --- | --- |
| Can harness changes help with weights fixed? | PRISM gains; Cursor operational metrics | PRISM small offline sets; retention regressions in EvoHarnessBench | Yes, with regression and cost controls. |
| Does an intrinsically stronger model remove harness sensitivity? | Some stronger models need fewer guardrails (Cursor) | Harness-Bench finds model–harness pairing effects; native tool contracts differ | Co-design for the actual model; no universal harness ranking. |
| Do extra prompts/agents improve reliability? | Directed prompt/middleware repairs help in PRISM | Auditor anchoring; unnecessary expensive summarization (Cursor) | Prefer failure-localized, testable changes and independent evidence paths. |
| Is an observed pattern a best practice? | Source-code audit finds common patterns | Observational sampling cannot identify causal performance effects | Use as hypotheses to test, not prescriptions. |
| Does the harness make a measurable, separable contribution? | MedicalHarness freezes models across five harnesses and reports ≈12% harness plus ≈12% model–harness interaction variance | 107 medical-oriented tasks; variance shares change by workload and are not universal | Evaluate model–harness–task pairings, not a global harness rank. |
| Is generic unit-test coverage adequate for agent harness code? | HarnessTester finds 122 bugs and outperforms baseline coverage/mutation gains on LLM-dependent harness paths | Reconstructed contracts can be wrong; selected repositories; coverage ≠ successful agent tasks | Use contract-faithful harness integration/testing and verify real task effects. |
| Do small decision models replace a large LLM without consequences? | Calibrated JevAgentHarness reports major savings in selected discrete-action settings | Uncalibrated delegation loses 19 points in one setting; coding agents save far less; auxiliary training and serving cost | Optimize routing per decision/workload; disclose all model updates and cost amortization. |
| Are model rankings stable across harnesses? | Prior Harness-Bench / MedicalHarness interactions | Finding the Right Fit finds sign-reversals and different cost frontiers; one run/task, nonuniform tools/budgets | Evaluate the full (model, harness, task, budget) configuration and confidence. |
| Is the cause of tool failure always poor reasoning? | Large menus under fixed cap lower ToolBottleneckBench success | Most penalty disappears when completion cap grows in six truncation-heavy settings | Diagnose termination and budget before changing model. |
| Does auto-approval simply increase productivity? | Legitimate utility +18.3 pp in HarnessSecurity-Bench | Attack success +66.4 pp, with alternatives through permitted paths | Evaluate utility and attack effects simultaneously under enforced policies. |
| Is a distinct verifier agent equivalent to independent verification? | VeriHarness improves selection and revision with workspace evidence | Shared consensus can be wrong; fixed-rollout and cache costs matter | Use independent evidence sources, not just majority or new roles. |
| Do more proof subagents necessarily help? | Maestro proves more Lean problems than native agents | Decomposition-only ablation near plain baseline, expensive global coordination | Test failure-aware replanning vs decomposition at equal total cost. |
| Do harness construction models implement full contracts reliably? | Doc2Harness deterministic contract tests isolate source artifacts | Best of 19 builder models only 53.05%; no downstream executor-task outcome evaluated | Test constructed harness code against behavioral contract plus positive controls. |

## Implementation/evaluation questions to track

- Per-model compatibility, **state and tool-call correctness**, rollback and model-switch semantics.
- Held-out changes after *multiple* consecutive harness updates; regressions on previously solved tasks.
- Task completion vs tool-call accuracy vs user satisfaction; cost per successful task, tokens, latency, tail reliability.
- Benchmarks that allow component-level causal isolation while preserving native agent-loop behavior.
- Contract-faithful tests for tool arguments, LLM response interpretation and asynchronous control flow; distinguish line/branch coverage, mutation score, and end-to-end safety.
- Independent model×harness×task variance estimates with confidence intervals, fixed executor checkpoints, paired task seeds, episode-level token/cost usage and failure diagnoses.
- Calibrated decision-model routing vs pure LLM execution under matched task success, GPU/training amortization, workload shifts and coding vs discrete-action domains.
- Paired benign/adversarial policy toggles with independent oracles and alternate allowed execution routes; compare LLM vs real human review.
- Completion/episode/tool-call budgets, truncation before first tool invocation, and tool-menu pressure; record both failure stage and diagnosed cause.
- Native and nonnative fixed-model harness pairings under equalized model/tool/cost budgets, matched task seeds and multi-run intervals.
- Contract-level scripted mock-executor tests plus positive controls for permissions, file operations, session persistence and invalid payloads.
- Evidence-grounded verifier selection+revision versus majority/best-of-N under matched **total** rollout + checking + repair spend.
- Failure-aware global replanning vs decomposition-only in verified and non-verified workloads; all-run cost and wall-time Pareto frontiers.

## Review status and retained history

2026-11-09: reviewed newly available 2026 publications and recovered six **previously published** but absent H1 evaluations: HarnessSecurity-Bench (Oct 6), ToolBottleneckBench (Oct 15), Finding the Right Fit (Oct 1), Doc2Harness (Oct 14), Maestro (Oct 19), and VeriHarness (Oct 1). These materially strengthen the policy security–utility, effective budget, configuration interaction, executable-harness contract testing, global coordination and independent verification conclusions. **None is a new Nov 2–9 H1 publication.** Nov 2 continual harness forgetting and Nov 4 HarnessFlow are primarily **H2**, so not imported as H1 evidence. No independent production replication verified. Parent/taxonomy/sibling files read-only and untouched.

2026-10-19: added and critically classified HarnessTester (testing), MedicalHarness (controlled variance/component ablation) and JevAgentHarness (learned auxiliary decision controller) with costs, protocols and contradictory results. The existing Barbaste et al. foundational anatomy and previous evidence remain intact.

First H1-focused synthesis assembled on 2026-10-14 from existing parent research and primary sources. No claim of a **new October H1 breakthrough** is made merely because this specialized file was initialized. Historical multi-agent/AgentOps work continues to reside in the **untouched** parent document until the human researcher integrates it.
