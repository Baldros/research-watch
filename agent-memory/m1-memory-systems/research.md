# M1 — Memory Systems & Retrieval — Living Research State

_Reviewed: 2026-11-09. Reconciles new primary evidence (R12–R16) with the 2026-10-26 baseline. Recent scores are author-reported; no independent replication was performed. Only this file is scheduler-owned; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files remain read-only._

## Current synthesis

1. **Strong original-history retrieval remains a mandatory control.** Zhou et al. [R7] found no consistent advantage for six explicit-memory systems over raw-history retrieval. New KITE [R12] demonstrates that typed, temporally constrained retrieval can beat BM25/dense/hybrid *over the same constructed memory store*. This is a different causal comparison: fixed-store retrieval gains do not show that its memory construction plus query compilation beats a tuned raw-history system with matched total cost.
2. **Access can be modeled as constrained evidence selection rather than top-k similarity alone.** KITE [R12] compiles typed lookup plans; EPGM [R9] groups agent execution events by their source provenance before retrieval. Both expose why evidence was selected. The index and query planner still incur ingestion, model-call, memory and latency costs.
3. **Persistence scope determines the best retrieval interface.** MemGPT [R1] proposed tiered paging; xMemory [R2] semantic/episodic hierarchies; EdgeMem [R3] evidence-preserving indexing; Mem++ [R4] time-attributed documents. Memorix [R13] adds immutable, version-pinned project views reused by coding agents; this differs materially from per-question conversational retrieval, and needs comparison with cached, version-filtered file search.
4. **Which experiences share a memory matters independently of representation.** Routed Continual Learning [R14] partitions interleaved task streams into memory-state clusters, selected by a *prompted, not trained* LLM router. Clustering accounts for most gains over a global history; choosing among alternative memory systems adds a smaller increment. A per-cluster 100k-token budget versus one global 100k budget prevents claims of total-capacity matching.
5. **A sufficient task state can outperform both long histories and naive RAG under specific conditions.** Guo & Guo [R16] provide a bounded, order-oblivious-retention impossibility result for a constructed known-rule event process and demonstrate that dependency-chain retrieval can match deterministic state filtering in one tested setting. This is not a universal theorem that open-world agents should discard history.
6. **No universal architecture winner emerges.** EvoMemBench v3 [R15] compares 15 memory methods across knowledge/execution and within-/cross-episode settings and finds long-context controls competitive in multiple regimes. MHG-Mem [R8] reports multimodal advantages with roughly twice comparator token consumption; EPGM's trained graph reranker [R9] has extra impact mainly on linked evidence, not simple facts. MemoryData [R5], LongMemEval [R6] and associative MRAgent [R11] provide distinct evaluation/architecture anchors.
7. **Safety and integrity cannot be inferred from retrieval accuracy.** Provenance loss, contradictory or stale entries, wrong library versions, failed erasure, cross-agent access and concurrent writes are distinct risks. The MemClaw/ArgusFleet [R10] direct-GET scoping defect was remediated; it remains a useful regression-test scenario, not a claim of a current vulnerability.

**Working hypothesis, not a validated unified system:** preserve inspectable original sources, compare retrieval against those sources first, then introduce constrained plans, hierarchy, version-pinned project views or clustered histories only when task-level gain survives matched token, storage, construction, compute and latency budgets. Enforce provenance and authorization on *every* access path.

## Evidence registry

### R1 — MemGPT: Towards LLMs as Operating Systems

- **Authors:** Charles Packer et al. **Date:** 2023-10-12; revised 2024-02-12. **Status:** research preprint. **Primary:** https://arxiv.org/abs/2310.08560
- **Mechanism:** the model manages a bounded active context while reading/writing information in larger external tiers, approximating virtual memory. Changes occur in harness-managed state; no retraining of executor parameters is required.
- **Evidence:** extended-document analysis and multi-session conversational use cases in the original paper.
- **Assessment:** foundational architecture, *not* proof the original MemGPT implementation is superior to 2026 baselines or survives restart/permissions checks without additional engineering.
- **Tags:** text/external; hierarchical paging; inter-session possible; M2 secondary.

### R2 — xMemory: Beyond RAG for Agent Memory: Retrieval by Decoupling and Aggregation

- **Authors:** Zhanghao Hu, Qinglin Zhu, Hanqi Yan, Yulan He, Lin Gui. **Date:** 2026-02-02. **Status:** arXiv preprint. **Primary:** https://arxiv.org/abs/2602.02007
- **Mechanism:** decouples correlated dialogue traces into semantic units, episodes and higher-level themes; retrieves diverse high-level units then expands to raw evidence when needed, counteracting redundant top-k selection.
- **Evidence:** author evaluations on LoCoMo and PerLTQA with multiple reader backbones; reported quality/token-efficiency gains.
- **Limitations:** memory construction and adaptive expansion require separate compute accounting; QA is not proof of autonomous task transfer; extraction can distort temporal context.
- **Tags:** hierarchical/structured; external episodic+semantic; M2 secondary for split/merge.

### R3 — EdgeMem: LLM-Free Agent Memory Construction and Retrieval via Evidence-Preserving Multi-Anchor Hypergraph

- **Authors:** Zeyang Cui et al. **Date:** 2026-09-03 (v1). **Status:** arXiv preprint. **Primary:** https://arxiv.org/abs/2609.05553
- **Mechanism:** keeps original interaction turns and organizes them via complementary content, time and episode anchors in a hypergraph; indexing/retrieval rely on lightweight local processing without *generative LLM calls*. The answer reader still uses an LLM.
- **Reported evidence:** under a shared LoCoMo answering prompt, strict-judge score **61.01 versus 58.70** for the strongest reproduced comparator among the reported seven systems; also tested on LongMemEval-S.
- **Cautions:** paper explicitly states its earlier public code differs substantially from the paper method, with a matching release promised after peer review. Claims of low overhead must still include local embedding/NLP, indexing, storage and retrieval costs. The tested conversational benchmarks do not demonstrate task execution, deletion or cross-agent trust.
- **Interpretation:** serious simple/low-generation baseline for evidence-preserving memory; not evidence against every abstraction or memory-learning approach.
- **Tags:** hypergraph + raw turns; non-generative construction; M2 secondary if retention is managed.

### R4 — Mem++: Non-Destructive Memory for Long-Term Organizational LLM Agents

- **Authors:** Ahmad Yehia et al. **Date:** 2026-10-01 (v1). **Status:** arXiv preprint. **Primary:** https://arxiv.org/abs/2610.02002 ; **evaluation code:** https://github.com/AIDAChip-Inc/mem-plus-plus
- **Mechanism:** stores whole dated/attributed documents rather than overwriting previous decisions; read-time temporal filtering plus lexical/semantic rank fusion. Old versions remain inspectable.
- **Reported evidence:** on OrgMemBench the authors report gains of **8.0–13.1 points** versus their strongest specialized-memory baseline, depending on reader, and **2.6 points** over their ordinary-RAG baseline for gpt-4.1-mini. Also evaluated LoCoMo and LongMemEval-S.
- **Cautions:** organization-specific evaluation does not establish general success across real corpora; document dates are not always precise effective/knowledge times. *Non-destructive retention is not the same as complying with a deletion request*. Inspect graph-variant temporal filtering separately rather than transferring claims from the base method.
- **Tags:** full document + temporal metadata; lexical/dense hybrid; M2 secondary for version validity.

### R5 — Are We Ready For An Agent-Native Memory System?

- **Authors:** Wei Zhou et al. **Date:** 2026-06-23. **Status:** arXiv preprint / systematic evaluation. **Primary:** https://arxiv.org/abs/2606.24775 ; **code:** https://github.com/OpenDataBox/MemoryData
- **Mechanism:** decomposes memory engineering into representation/storage, extraction, retrieval/routing and maintenance; tests **12 representative systems and two reference baselines** across five benchmark workloads (11 datasets).
- **Key finding:** no one memory architecture wins on every workload; local maintenance can be less expensive than global reorganization. This supports workload-level selection and component ablations.
- **Limitations:** do not assume every system was evaluated in every cost experiment; outcomes depend on benchmark, judge, model and harness. Their paper's module numbering is **unrelated to this repository's M1–M4 research taxonomy**.
- **Tags:** evaluation; M2/M3/M4 interfaces.

### R6 — LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory

- **Authors:** Di Wu et al. **Date:** 2024-10-14, v2 2025-03-04. **Status:** **ICLR 2025**. **Primary:** https://arxiv.org/abs/2410.10813
- **Mechanism:** 500 questions over scalable conversation histories, testing information extraction, multi-session and temporal reasoning, knowledge updates, and abstention. Splits the pipeline into **indexing / retrieval / reading**.
- **Interpretation:** evaluation anchor, not a new memory method. Use the precise dataset subset/version; a higher QA score does not imply better deletion, safe sharing, real-world agent performance or portable procedural skills.

### R7 — Do Conversational Agents Need Explicit Memory? A Controlled Comparison with Raw-History Retrieval — high-priority counterevidence

- **Authors:** Yongjie Zhou, Shuai Wang, Bevan Koopman, Guido Zuccon. **Submitted:** 2026-10-17; **status:** arXiv v1 preprint / comparison. **Primary:** https://arxiv.org/abs/2610.24671
- **Design:** six explicit-memory systems (MemoryBank, A-Mem, Mem0, LightMem, SimpleMem, HyperMem) compared with raw-history retrieval over original turns, user turns, and turn pairs. LongMemEval-S has **500** questions, BEAM-100K **400**. They vary retriever/budget and include oracle supporting evidence to distinguish memory-construction loss from retrieval failure.
- **Reported results:** native LongMemEval-S configurations give **78.0% for all-turns RAG using Qwen3-Embedding-0.6B vs 64.0% LightMem**, and BEAM **56.2% raw turns with MiniLM vs 53.9% MemoryBank**. **Important:** these initial rows do not match every retrieval/answer component. At a common **1,000-token** setting, the best *per question type* performance averages **73.2% raw-history vs 69.0% explicit** on LongMemEval-S and **57.7% vs 54.4%** on BEAM. These are question-type selection summaries, **not** deployable scores for one single system.
- **Why this matters:** several systems preserve evidence but rank it poorly; others delete information during condensation. In BEAM, an oracle-to-practical retrieval gap of roughly **21–22 points** was reported for some explicit-memory variants. Constructing LightMem and A-Mem cost about **6.2 and 215.1 minutes per conversation** respectively in the paper's particular H100 setup; these are *construction*, not per-query, timings.
- **Limits:** not all systems use unchanged defaults; only six specified methods were evaluated, not xMemory/EdgeMem/Mem++. QA workloads cannot establish tool-use success, restart durability, privacy deletion or multi-agent scoping. Raw retrieval itself has indexing/embedding/storage cost. Native/controlled conditions must not be mixed as one ranking.
- **Assessment:** **weakens universal superiority claims for explicit extracted memory** and strengthens raw-history retrieval as a serious baseline. Does not establish that structured memory is never useful. **M1 primary, M2 secondary for loss during writing.**

### R8 — Remembering Through Change: Provenance-Preserving Hypergraph Memory for Multimodal Agents (MHG-Mem)

- **Authors:** Jielong Tang, Ling-An Zeng, Xujie Yuan et al. **Submitted:** 2026-10-21; **status:** arXiv v1 preprint. **Primary:** https://arxiv.org/abs/2610.29995
- **Substrate/access:** external multimodal observations, entity mentions and evolving facts linked by occurrence, evolution and evidential hyperedges. The agent selects memory layers and traverses links to retrieve text and original visual sources; source pointers support evidence chains. No executor-weight training is established by this mechanism.
- **Reported evidence:** three benchmarks (**MemEye, Mem-Gallery, WorldMemArena-S**), three readers (**Qwen3-VL-8B, GPT-4.1-mini, Qwen3.8-27B**), eight reproduced baselines, and three runs. Authors report **8.20–9.98 point** aggregate-judge gains over their strongest compared system across reader settings. Reported ablations indicate inspecting source observations, traversal and iterative search matter.
- **Limits:** roughly **2× token consumption** relative to tested multimodal competitors. Matching reader does not imply equal tokens, latency, storage, visual encoding or search effort. No independent cross-agent authorization or privacy-erasure audit.
- **Assessment:** supports provenance-aware multimodal state retrieval in its tested conditions, without contradicting [R7]'s text-conversation result. **M1 primary, M2 secondary for temporal evolution.**

### R9 — EPGM: Execution Provenance for Budgeted Agent Memory Retrieval

- **Authors:** Yiqi Wang, Jinqian Ju, Jiaqi Zhang et al. **Submitted:** 2026-09-22; revised **2026-10-06 (v2)**; **status:** arXiv preprint. **Primary:** https://arxiv.org/abs/2609.25913
- **Substrate:** source-aligned spans from *tool-execution histories*, grouped into provenance-respecting candidate units rather than arbitrary overlapping text windows; optional **trained residual R-GCN** propagates scores along typed edges. Main novel retrieval substrate/access is **M1**; learnable reranker is **M3 secondary**. Trained R-GCN ≠ trained executor.
- **Reported evidence:** **2,000 span-grounded questions** over **1,207 held-out trajectories**, with **2,048-token** full-support retrieval budgets. Compared with 512-token flat windows, provenance grouping improves Full Support@2048 by **19.07 points**; holding candidates and seed scores fixed, graph propagation adds **4.55 points** (95% CI **[2.98, 6.18]**). Linked/multi-fact queries benefit more than direct facts; learned propagation adds only about **0.19 points** on the direct-fact split.
- **Downstream qualification:** a smaller 315-QA cohort reports roughly **+6.03 points strict-answer accuracy** over flat retrieval but with 95% CI **[0.00, 12.10]**, so the end-to-end result is borderline. Reported inference overhead of graph propagation is ~**20 ms / 11 MiB**, excluding training and full memory system construction.
- **Assessment:** source-aligned grouping has strong measured retrieval benefit; typed graph propagation adds selective value when evidence is distributed. Relevance outside execution traces, persistence after restart and security remain open.

### R10 — Governed Shared Memory for Multi-Agent LLM Systems — engineering and security counterexample

- **Authors:** Yanki Margalit, Nurit Cohen-Inger, Erni Avram, Ran Taig, Oded Margalit. **Submitted:** 2026-06-23; **status:** arXiv preprint / authors' service evaluation. **Primary:** https://arxiv.org/abs/2606.24535 ; [test harness](https://github.com/caura-ai/argusfleet).
- **Mechanism:** the MemClaw shared-memory service has agent/fleet/tenant scopes, semantic search and direct GET-by-id, provenance chains, temporal supersession and visibility rules. No executor training is needed for these external-state access semantics.
- **Specific negative finding:** an earlier **trust-level-1 credential could fetch cross-fleet same-tenant memories by ID** because direct GET authorization did not enforce the same agent/fleet scope as intended. The issue was disclosed and **remediated**; the authors' follow-up reported **0/36** unauthorized trust-1 cross-fleet direct reads versus **36/36** authorized trust-2 reads. This does **not** assert an unpatched present vulnerability.
- **Consistency/latency:** a near-duplicate write gate could reject conflicting entries before asynchronous resolution; conditional supersession success does not measure all write conflicts. Under tested load, p99 write latency rose substantially; measure temporal visibility and concurrency alongside search accuracy.
- **Limits:** self-evaluation with narrow tenant/permission matrix and no matched alternative backend or independent audit. Production behavior cannot be extrapolated beyond tested conditions.
- **Assessment:** every direct lookup, semantic search, graph traversal and cache route must enforce the same scope rules. **M1 primary for authorized access; M2 secondary for lifecycle correctness.**

### R11 — Memory is Reconstructed, Not Retrieved: Graph Memory for LLM Agents (MRAgent)

- **Authors:** Shuo Ji, Yibo Li, Bryan Hooi. **Submitted:** 2026-06-04; **status:** ICML 2026 acceptance reported by authors/arXiv. **Primary:** https://arxiv.org/abs/2606.06036
- **Mechanism:** Cue–Tag–Content associative graph accessed through active LLM-driven exploration, pruning and expansion of supporting context rather than one retrieve-then-answer pass; external state can change without executor weight updates.
- **Reported evidence:** LoCoMo and LongMemEval gains **up to 23%** versus selected comparators with lower measured token/runtime under their conditions.
- **Limits:** iterative decisions add potential control/tool cost; benchmark answer quality does not prove persistent process restart, general downstream task transfer, complete deletion or access isolation.
- **Assessment:** relevant evidence for *adaptive associative retrieval*. Compare fairly with raw-history [R7], source-provenance units [R9] and matched reader/search budgets.

### R12 — KITE: Retrieval as a Decision Procedure for Long-Term Conversational Memory

- **Authors:** Feng Li, Kailun Tan, Zhibo Zhou, Youxiang Wang, Lejun Liu, Huaigu Li, Fei Du, Huilong An. **Submitted:** 2026-10-30; **status:** arXiv v1 preprint. **Primary:** https://arxiv.org/abs/2610.41521 ; **full text:** https://arxiv.org/html/2610.41521v1 ; **code:** https://github.com/memoket/memoket-kite
- **Persisted state / access:** multi-session dialogue distilled into source-linked structured entries with governed topics, entity/participant/event/place facets and temporal metadata. An LLM compiles each question into typed \`select/where/pipe\` subqueries, validated against the index; posting-list operations and ranking execute deterministically per validated plan, with sparse lexical fallback. No learned executor or memory-policy weights reported: external memory/index and prompts evolve.
- **Author-reported evidence:** **1,540 LoCoMo questions** (ten conversations) and **500 LongMemEval-S** questions. KITE obtains **93.5%/86.0%** LLM-judge accuracy with roughly **1.5k–1.6k answer-reader evidence tokens**. Importantly, with the *constructed store held fixed*, KITE versus hybrid retrieval scores **93.5 vs 87.8** on LoCoMo and **86.0 vs 78.0** on LongMemEval-S; evidence recall **87.4 vs 79.9** and **97.6 vs 92.8**, respectively. This isolates retrieval strategy better than comparisons across different memory constructors.
- **Limits:** write-time generative extraction, multiple candidate query-compilation calls and initial benchmark-aware topic roots have additional costs; equal *reader context* is not equal total tokens, storage or latency. The reader and evaluator both use **gpt-4.1-mini**. LongMemEval-S single-session preference accuracy is only **63.3%**. No strong fully matched raw-history comparison [R7], independent repetition, restart/permission, poisoning or deletion audit.
- **Assessment:** credible evidence for *constraint-respecting retrieval over a fixed indexed memory*, not universal dominance over vector search or raw evidence retrieval. **M1 primary; M2 secondary** for vocabulary maintenance.

### R13 — Memorix: Managing Memory Selections for AI Agents

- **Authors:** Jing Zhang, Xiaguannan Song, Dong Du, Erhu Feng, Yubin Xia. **Submitted:** 2026-10-13 (arXiv **2611.00060**); **status:** v1 preprint. **Primary:** https://arxiv.org/abs/2611.00060 ; **full text:** https://arxiv.org/html/2611.00060v1
- **Persisted state / access:** **SHA-256 content-addressed immutable blobs**, versioned trees, reproducible \`Derivation\` selections assembled into project-level \`Environment\` views, and \`Quest/Profile\` update interfaces. A coding agent accesses a precomputed mounted project view using ordinary file tools. No neural training. Historical blobs remain recoverable after pathname deletion: **versioning is not erasure**.
- **Author-reported evidence:** on **99 SWE-ContextBench Lite tasks** from 12 projects with **300 historical trajectories**, same DeepSeek-v4.1-flash/Pi agent, resolved **64.6%** (Memorix) vs **59.6%** MemOS, **57.6%** OpenViking, **50.5%** Mem0, **44.4%** no-memory. On **212** version-conditioned library tasks, **68.4%** vs **60.4%** Mem0. At 99 reused tasks, amortized API cost per task: **$0.099 Memorix vs $9.94 MemOS, $0.498 Mem0, $0.089 no-memory**; the approximately 100x figure is **versus MemOS**, not versus no memory.
- **Limits:** competing memory systems retrieve online through system-specific plugins; Memorix gets a preselected mounted view, and the library-version benchmark supplies target version information. Compare against cached, version-filtered file/RAG views on equal project-reuse, cold-start, storage and latency budgets. No compelling multi-tenant concurrency, revocation or privacy-deletion evaluation.
- **Assessment:** meaningful **version-correct project-context persistence**, not evidence against all query-time retrieval. **M1 primary; M2 secondary** for version validity.

### R14 — Routed Continual Learning

- **Authors:** Michael Philip, Mahesh Balakrishnan, David Geraghty, David Mao, Vidhya Venkat, Ilya Mironov. **Submitted:** 2026-10-28; **status:** arXiv v1 preprint. **Primary:** https://arxiv.org/abs/2610.38796 ; **full text:** https://arxiv.org/html/2610.38796v1
- **Persisted state / routing:** a prompted **LLM router** chooses whether to reuse/create a history cluster and selects among five external memory representations, including raw trajectories, extracted strategies, hierarchical regression and task-state estimators. A Context Memory Adapter mediates bound memory access. Agent/executor weights are frozen; choosing a route via prompts is **not M3-trained policy learning**.
- **Author-reported evidence:** **398 interleaved tasks** (301 CL-Bench, 97 banking), four readers and five paired rollouts. Global ICL vs routed ICL-only vs Routed CL separates clustering from representation choice. Routed CL improves normalized reward by about **0.15–0.30 of the maximum possible reward** relative to global ICL while reducing estimated API cost **65–72%**. Isolated memory-system selection adds only **0.002–0.076 reward points** above routed ICL; its incremental benefit is model/task-dependent.
- **Limits:** Global ICL has one **100k context limit**, routed ICL one **100k per cluster**: *total retained state is not matched*. Candidate systems were selected using exploratory CL-Bench exposure; genuine unseen-workload transfer, privacy/authorization and poisoning are untested. Routed CL can cost more than some single-system alternatives.
- **Assessment:** evidence that **partitioning shared experience is a first-class access decision**, distinct from selecting a storage representation. **M1 primary; M4 secondary** for experience reuse.

### R15 — EvoMemBench: Benchmarking Agent Memory from a Self-Evolving Perspective (v3)

- **Authors:** Yuyao Wang, Zhongjian Zhang, Mo Chi, Kaichi Yu et al. **Submitted:** 2026-05-18; **v3 revised:** 2026-10-30; **status:** **accepted NeurIPS 2026**, per arXiv. **Primary:** https://arxiv.org/abs/2605.18421 ; **v3 full text:** https://arxiv.org/html/2605.18421v3 ; **code/data:** https://github.com/DSAIL-Memory/EvoMemBench
- **Role:** **evaluation benchmark, not a proposed new memory substrate**. Tests the 2×2 combinations of **within-/cross-episode** retention and **knowledge-/execution-oriented** tasks, comparing **15 memory methods** and long-context controls. Cross-episode state is updated within an environment group, then reset between independent groups.
- **Evidence and caution:** no representation dominates; long-history baseline often remains competitive, especially where existing context suffices; retrieved knowledge can inject irrelevant or stale facts, and procedural memory helps where accumulated skills match future execution. Memory-augmented conditions share **DeepSeek-V3.2**, but some long-context reference rows use **Gemini-3-Flash/GPT-5-mini**; never treat those as same-executor comparisons. Token accounting includes memory-related model calls but does not fully match construction, training, storage, tool latency or operational security.
- **Assessment:** useful cross-domain corrective to one-benchmark leaderboards, especially for stale memory and task transfer. **M1 evaluation anchor; M2/M4 cross-tags**, not evidence that one M1 retrieval method has solved all tasks.

### R16 — Agent Memory as Rate-Limited Filtering: Explicit State Outlasts Long Context Beyond a Reader-Dependent Horizon

- **Authors:** Tianhao Guo, Chongyao Guo. **Submitted:** 2026-11-02; **status:** arXiv v1 preprint. **Primary:** https://arxiv.org/abs/2611.02361 ; **full text:** https://arxiv.org/html/2611.02361v1
- **Substrate / theoretical claim:** a known-rule access-control event stream has a recursively maintainable finite-key **sufficient state**; LLMs parse events and deterministic rules apply ordered updates. For a specified counterfactual chain and **order-oblivious** extractive retention of \`b\` lines from \`m\` relevant events, optimal reader error satisfies **max(0, 1/2 − b/(m−1)) ≤ Err* ≤ 1/2**. This does *not* rule out dependency-aware or order-sensitive retrieval and is **not** an open-world universal compression theorem.
- **Reported evidence:** Qwen2.5-14B full-history performance degrades substantially by 256 events. At **1,024 events**, gpt-oss-120b reaches **0.87 full log vs 0.99 state built from parsed events**; event parsing plus deterministic rules obtains **0.80–0.89** at 256 events for tested readers. Crucially, **query-time retrieval of the actual dependency chain matches filtering** at 256 events on the first domain, challenging the simplistic claim that explicit write-time compression is inherently necessary.
- **Limits:** synthetic, finite-key policy processes; assumes accurate rules and known state schema; write cost includes a model call to parse *each* event. Contradictions, access-control policy drift, unknown schemas, cross-agent authorization and total-cost matching need more tests. The failure horizon changes with reader strength.
- **Assessment:** unusually clear conditional mathematical and empirical demonstration of **ordered state/dependency retrieval**, complementary to R7's raw-history evidence. **M1 primary; M2 secondary** for temporal state transitions.

## Assessment matrix / experiment design

| Claim under test | Essential controlled comparisons |
| --- | --- |
| Explicit extracted memory beats raw history | Strong raw all-turns, user-only and turn-pair lexical+dense retrieval, same answer model/prompt/context and full index+write+read cost [R7] |
| Graph/hierarchy outperforms RAG | Same executor, corpus, temporal ground truth, top-k and *actual context-token* budget; lexical+dense baseline |
| Did the memory constructor lose evidence? | Oracle support *before and after* construction versus practical retriever on the stored substrate [R7] |
| Do typed plans improve retrieval? | Fix stored entries, reader and evidence cap; compare compiled typed plans with BM25, dense/hybrid, full-history, counting compilation/ingestion cost [R12] |
| Do persistent project views beat search? | Equal original files/version labels, cached version-filtered lexical+dense retrieval, tasks per project, storage/cold-start and total latency [R13] |
| Is cluster routing the true driver? | Global ICL, clustered ICL-only, multi-substrate routed memory with equal **total retained state** (not only per-cluster limits), model/task and stream [R14] |
| Does explicit state beat dependency-aware retrieval? | Same policy stream, correct rule, oracle state, full ordered log, BM25, exact dependency-chain query retrieval; amortize per-event parser/transition cost [R16] |
| Is graph advantage from units or edges? | Hold candidate units and dense seeds fixed; ablate source grouping, trained graph propagation and generic cooccurrence [R9] |
| Do visual evolution links justify overhead? | Same reader and multimodal observations with matched image tokens, search calls, total latency and budgets [R8] |
| Source preservation prevents forgetting | Exact evidence retention after update, retrieval@budget and answer correctness after long delays |
| Ingestion is cheap | Count embedding/index passes and LLM calls at write **and** read time; include full storage footprint |
| Temporal retrieval is valid | Current-state versus as-of queries; prevent using evidence not yet available at the asked time |
| Multi-session persistence works | Stop and restart process; reconstruct index, authorization and retrieval behavior |
| Shared memory is safe | Enforce and test scope on **all** read routes, including direct GET-by-id, graph traversals and caches [R10]; injection and deletion cross-link M2/security |
| Multi-agent writes remain consistent | Concurrent conflicting writes, near-duplicate gating, supersession acceptance, read-after-write visibility and p95/p99 latency [R10] |

**Quality interpretation:** R6 is an ICLR 2025 benchmark, R11 reports ICML 2026 acceptance and R15 v3 reports NeurIPS 2026 acceptance. R12–R14 and R16 are preprints with largely author-run experiments; none here has been independently replicated for this review. R7's raw-history control is different from R12's fixed-constructed-store retrieval ablation. R13 reuses precomputed project context while its comparators retrieve online; R14 allows separate 100k-token limits per cluster; R16 proves a result only for a defined finite-key, order-oblivious-retention class. Do not merge incompatible benchmarks, reader/judge scores and budgets into a universal leaderboard.

## Open work and next research priorities

- **Highest priority:** reproduce strong raw-history retrieval [R7] against KITE's typed plans [R12] and task dependency-aware retrieval [R16], fixing executor, answer prompt, true source history, oracle support, reader-context tokens and **total** write/index/compile/read/storage/compute/latency budgets. Distinguish construction loss from retrieval failure.
- Compare source-preserving [R3, R4] and generative-extraction [R12] stores under temporal updates, contradictory facts, adversarial distractors, unseen query types, and multi-session process restarts.
- Reproduce Memorix [R13] with cold-start and **cached version-filtered file-search** baselines as the number of tasks per project changes; quantify old-version contamination, storage growth, permissions and complete deletion behavior.
- Re-test Routed CL [R14] with equal **aggregate memory retention** across all clusters, not only per-cluster context, and with held-out task families, multi-agent identity boundaries, state poisoning and cross-agent sharing.
- Use EvoMemBench v3 [R15] to separately assess knowledge retrieval, within-session state, temporal revision, execution and cross-episode transfer; ensure every direct comparison uses the same reader.
- Isolate source-aligned grouping vs trained graph propagation [R9] on new tool-execution traces; match text/image tokens and search/model costs when revisiting multimodal evidence [R8].
- Maintain a **memory observability** audit: source provenance, validity/effective time, version, read trace, contradictory/stale status, agent/tenant scope, delete propagation and visibility after process restart.
- Verify that GET-by-id, semantic retrieval, graph traversals, caches and subscriptions apply the same authorization policy; use [R10]'s *resolved* scope defect as a regression test, not a current vulnerability assertion.
- Cross-link learned memory retrievers to [M3](../m3-learned-memory/research.md) rather than treating prompted retrieval as training; M2 owns entry lifecycle and M4 owns reusable skills.

_Review continuity: 2026-10-20 R1–R6; 2026-10-26 R7–R11; 2026-11-09 R12–R16. In-place synthesis, not an append-only run log. This review verified primary papers but did not execute benchmark reproductions, security audits or process-restart tests._
