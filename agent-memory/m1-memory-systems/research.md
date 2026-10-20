# M1 — Memory Systems & Retrieval — Living Research State

_Reviewed: 2026-10-20. Initial evidence-backed research pass, including foundational and 2026 publications. This file alone is scheduler-owned; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files are read-only._

## Current synthesis

1. **Architecture and access are distinct from window length.** Durable agent memory requires a representational substrate, persisted state, and read/query policy. MemGPT introduced a useful context/storage-tier abstraction; modern approaches test structured retrieval (xMemory) and non-generative evidence preservation (EdgeMem). None proves a universal best representation. [R1–R3]
2. **Retaining original evidence can matter more than generating summaries at ingestion.** EdgeMem preserves turns and non-generatively indexes them; Mem++ preserves dated source documents and shifts selection to read time. Their evidence challenges the assumption that write-time LLM distillation is always necessary, but does not disprove context compression in all regimes. [R3–R4]
3. **No architecture dominates every workload.** A broad comparative study finds workload-sensitive performance and cost trade-offs among memory systems. Judge prompts, subsets, reader identity, ingestion costs and history length can change leaderboard results. [R5–R6]
4. **An evidence-preserving store is not automatically correct or safe.** Future fact leakage, outdated records, cross-user retrieval, and incomplete deletion need explicit testing. Physical deletion and privacy compliance belong primarily to [M2](../m2-memory-lifecycle/research.md).

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

## Assessment matrix / experiment design

| Claim under test | Essential controlled comparisons |
| --- | --- |
| Graph/hierarchy outperforms RAG | Same executor, corpus, temporal ground truth, top-k and *actual context-token* budget; lexical+dense baseline |
| Source preservation prevents forgetting | Exact evidence retention after update, retrieval@budget and answer correctness after long delays |
| Ingestion is cheap | Count embedding/index passes and LLM calls at write **and** read time; include full storage footprint |
| Temporal retrieval is valid | Current-state versus as-of queries; prevent using evidence not yet available at the asked time |
| Multi-session persistence works | Stop and restart process; reconstruct index, authorization and retrieval behavior |
| Shared memory is safe | Access scope, provenance, write conflicts, injected instructions, deletion semantics (cross-link M2/security) |

**Quality interpretation:** R6 is a peer-reviewed benchmark; R1 is foundational; R2–R5 are preprints and author-run evaluations. Do not collapse incompatible judge protocols, readers and dataset variants into a numeric ranking.

## Open work and next research priorities

- Compare **source-preserving vs generative-extraction** representations on matched ingestion+query budgets and mutable facts.
- Evaluate graph/hierarchical retrieval under high redundancy, adversarial distractors and real cross-session resets.
- Build a *memory observability* checklist: entry provenance, update history, read trace, permission scope, contradiction status, original-source link and deletion propagation.
- Cross-link trainable retrieval readers to [M3](../m3-learned-memory/research.md) instead of duplicating Memorilla as an M1-owned paper.

_Last review used primary published abstracts and supporting primary pages; no benchmarks were independently re-run. Next update should revise assessments in place, not append an execution log._
