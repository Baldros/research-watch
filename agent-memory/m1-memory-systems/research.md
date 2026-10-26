# M1 — Memory Systems & Retrieval — Living Research State

_Reviewed: 2026-10-26. New evidence reconciled with the 2026-10-20 baseline. Recent results are author-reported and mostly preprints. This file alone is scheduler-owned; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files are read-only._

## Current synthesis

1. **A strong raw-history retriever is a mandatory control, not a strawman baseline.** Zhou et al. [R7] find no consistent overall advantage of six tested explicit-memory systems over retrieval from original turns on LongMemEval-S and BEAM-100K. The study does **not** cover EdgeMem, xMemory, Mem++, or multimodal agent memory, and native systems differ in prompts and retrievers; do not extrapolate to all memory architectures.
2. **Persistent representation and evidence access have separate failure modes.** A constructor may erase relevant evidence; alternatively, an intact store may be poorly searched. Oracle evidence and a fixed reader help distinguish them [R7]. Structured candidate units that preserve source provenance can improve *complete* support retrieval under a fixed token budget [R9].
3. **Graph and hierarchy have conditional value.** MHG-Mem [R8] reports gains for evolving visual facts at roughly twice the token use of multimodal comparators. EPGM [R9] finds much of the improvement from source-aligned execution-event grouping, with an additional, smaller benefit from trained typed-graph propagation on linked evidence. Neither establishes a universally superior graph store.
4. **Retaining original evidence can be more robust than compulsory write-time summarization.** EdgeMem [R3] preserves raw turns, Mem++ [R4] preserves dated documents; both challenge the assumption that generative extraction is always required. Their advantages must be compared against strong raw-source RAG and full ingestion/retrieval cost.
5. **Shared memory needs authorization and concurrency semantics at every access path.** The MemClaw/ArgusFleet study [R10] reports an agent/fleet scoping defect in direct GET-by-id, later patched and re-tested. Retrieval is not safe simply because semantic search is filtered; this is a bounded system-specific finding, not a claim of an ongoing vulnerability.
6. **No architecture dominates across workloads.** MemGPT [R1] remains a tiered-context foundation; xMemory [R2] explores hierarchies; MemoryData [R5] and LongMemEval [R6] expose variable workload/judge effects; MRAgent [R11] adds a peer-reviewed example of iterative associative reconstruction.

**Working hypothesis, not a validated unified design:** preserve auditable source evidence, evaluate raw-history retrieval before adopting extraction, add structured indexes where they deliver budget-matched improvements, and enforce permission/provenance at every memory operation.

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

## Assessment matrix / experiment design

| Claim under test | Essential controlled comparisons |
| --- | --- |
| Explicit extracted memory beats raw history | Strong raw all-turns, user-only and turn-pair lexical+dense retrieval, same answer model/prompt/context and full index+write+read cost [R7] |
| Graph/hierarchy outperforms RAG | Same executor, corpus, temporal ground truth, top-k and *actual context-token* budget; lexical+dense baseline |
| Did the memory constructor lose evidence? | Oracle support *before and after* construction versus practical retriever on the stored substrate [R7] |
| Is graph advantage from units or edges? | Hold candidate units and dense seeds fixed; ablate source grouping, trained graph propagation and generic cooccurrence [R9] |
| Do visual evolution links justify overhead? | Same reader and multimodal observations with matched image tokens, search calls, total latency and budgets [R8] |
| Source preservation prevents forgetting | Exact evidence retention after update, retrieval@budget and answer correctness after long delays |
| Ingestion is cheap | Count embedding/index passes and LLM calls at write **and** read time; include full storage footprint |
| Temporal retrieval is valid | Current-state versus as-of queries; prevent using evidence not yet available at the asked time |
| Multi-session persistence works | Stop and restart process; reconstruct index, authorization and retrieval behavior |
| Shared memory is safe | Enforce and test scope on **all** read routes, including direct GET-by-id, graph traversals and caches [R10]; injection and deletion cross-link M2/security |
| Multi-agent writes remain consistent | Concurrent conflicting writes, near-duplicate gating, supersession acceptance, read-after-write visibility and p95/p99 latency [R10] |

**Quality interpretation:** R6 is a peer-reviewed benchmark; R11 reports ICML 2026 acceptance; R1 is foundational. R2–R5 and R7–R10 are preprints and mostly author-run evaluations. R7's six systems do not include R2–R4; R8's multimodal judge scores are not comparable with R7's text QA accuracy; R9 separates retrieval completeness from a borderline downstream answer gain. Do not turn incompatible scores into a universal leaderboard.

## Open work and next research priorities

- **Highest priority:** independently rerun [R7]'s **raw-history vs explicit-memory** comparisons with common reader, query/token budget, strong embeddings, total write+read accounting and oracle evidence after construction.
- Compare **source-preserving vs generative-extraction** representations on matched ingestion+query budgets and mutable facts.
- Distinguish source-aligned grouping from *trained* graph propagation [R9] on new event traces, and separately test provenance-aware multimodal queries at matched visual token/search budgets [R8].
- Evaluate graph/hierarchical retrieval under high redundancy, adversarial distractors and real cross-session resets.
- Build a *memory observability* checklist: entry provenance, update history, read trace, permission scope, contradiction status, original-source link and deletion propagation.
- Audit every shared-memory access route—GET-by-id, semantic retrieval, graph traversal, subscriptions and caches—using [R10] as a **resolved-scope-defect test case**, not a live vulnerability assertion.
- Cross-link trainable retrieval readers to [M3](../m3-learned-memory/research.md) instead of duplicating Memorilla as an M1-owned paper.

_This update incorporates R7–R11 into a living synthesis, preserving R1–R6. The technical claims above reflect author-reported versions/experiments; no benchmark, security probe, or independent replication was conducted in this review. Future updates should follow independent replications and substantive revisions, not simply append literature logs._
