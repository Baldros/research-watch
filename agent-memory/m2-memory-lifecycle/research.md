# M2 — Memory Lifecycle & Evolution — Living Research State

_Last reviewed: 2026-11-03. New M2 evidence integrated with the 2026-10-20 baseline; all recent quantitative results are author-reported. This scheduler owns only this file; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files remain read-only._

## Current synthesis

1. **Forgetting, correction, consolidation and deletion are different operations.** A low retrieval score is not deletion; merging two memories is not proof that both underlying sources remain accessible; rewriting an inaccurate reflection is not a change to frozen model parameters. The storage object and its provenance must be tracked through each operation.
2. **Two positive directions should not be collapsed.** FadeMem reports retrieval/reasoning gains with less storage through adaptive decay and fusion; ReHarness reports benefits from *backward revision* of prior textual reflections as online evidence arrives. Their evidence supports distinct mechanisms and does not establish that their combination is optimal. [L1–L2]
3. **A strong counterexample challenges automatic consolidation.** Zhang et al. find that continuously rewritten memories can become actively harmful, even when generated from useful trajectories; episodic-only retention can remain competitive. This weakens any blanket assumption that consolidation should fire after each event. [L3]
4. **Forgetting on user request requires more than suppressing retrieval.** MemCut [L4] and AfterDeleteBench [L5] show that derived evidence and summaries can reveal a supposedly deleted fact. AfterDeleteBench adds matched never-exposed controls and separately measures disclosure, re-storage and fresh-session recovery. Physical removal of one row and query-time refusal do not prove a fact has vanished from every dependent memory object.
5. **Validated consolidation is plausible but not universally safe.** ProactMem [L6] checks source-linked trajectories and candidate behavioral regularities against prior events before promotion. This is more selective than forced consolidation, but tested event streams do not establish robustness under open-ended preference drift, rare crucial events or unknown future tasks.
6. **Reversibility and deletion are fundamentally different.** MoM/P-Mem [L7] maintains an active version with a provenance history; ChronoMem [L8] snapshots state for semantic rollback. Both preserve historical information precisely because they support undo; neither, by itself, constitutes user-requested erasure.
7. **Revocation can also require rebuilding live execution state.** Forgetting Without Restarting [L9] addresses contaminated summaries, plans and runtime caches after an observation has been consumed. Reconstructing a counterfactual clean suffix is materially stronger than deleting a persistent record, but relies on replay and provenance assumptions.

## Lifecycle operation contract — working analytical framework

| Operation | Concrete mutation to audit | Misleading proxy to reject |
| --- | --- | --- |
| Write/select | Adds or rejects a fact/episode plus source provenance | Model saying it will remember |
| Consolidate | Adds or revises abstraction, with links to source events | Shorter prompt or fewer records alone |
| Revise | Edits versioned record, reflecting new feedback/conflict | Appending a new contradictory summary |
| Retain/decay | Updates importance/strength; a separate threshold or retention policy determines actual pruning | Lower priority falsely described as physical erasure |
| Forget/delete | Removes original and traced derived records or rebuilds contaminated summaries/indexes; audits remaining inference paths | Failure of normal top-k retrieval alone |
| Temporal supersede/contest | Commits current value with authority, validity and provenance, retaining displaced versions if rollback is intended | Overwriting a conflicting fact without a record of source evidence |
| Rollback | Restores a versioned memory state and correctly scopes all subsequent reads | Merely prompting the agent to ignore later facts |
| Runtime revocation | Reconstructs dependent plans, summaries, prompts and cached state after removal where replay is feasible | Assuming a persistent-store delete undoes the effect of a previously read fact |
| Expire/temporal correction | Distinguishes valid-at, known-at and supersession | Silently replacing history with a current fact |

This is a proposed engineering/evaluation vocabulary, **not a verified common implementation** of all cited systems.

## Primary evidence

### L1 — FadeMem: Biologically-Inspired Forgetting for Efficient Agent Memory

- **Lei Wei, Xiao Peng, Xu Dong, Niantao Xie, Bin Wang.** Submitted **2026-01-26**, revised **2026-02-06** (v2); arXiv preprint. [Primary source](https://arxiv.org/abs/2601.18642).
- **Mechanism:** dual-layer memory, differential adaptive exponential decay depending on relevance, access frequency and temporal behavior; LLM-guided conflict handling and memory fusion. Decay is applied to external agent-memory retention, **not** backpropagation into the executor weights.
- **Reported evidence:** Multi-Session Chat, LoCoMo and LTI-Bench; the authors report stronger multi-hop retrieval/reasoning with **45% lower memory storage** in their tested regime.
- **Implementation distinction:** the retention strength can decay exponentially, while semantic relevance/access/recency also contribute to importance. The paper describes layer promotion/demotion and threshold/dormancy-based **pruning**, as well as LLM-guided fusion and conflict suppression. Downranking, pruning, suppressing old strength and rewriting a merged summary are *different physical actions*—none proves complete deletion across indexes/backups.
- **Ablations reported by authors:** on LoCoMo, multi-hop F1 **29.43 full**, **13.63 without fusion**, **22.88 without conflict resolution**, **19.45 without dual layers**. These are not an independently reproduced, total-cost-matched study.
- **What to verify:** exact consequences of pruning, how merging treats source evidence, rare decisive memories and budget-matched LRU/LFU/TTL, summary and raw-history baselines including write+read LLM costs.
- **Status:** promising *preprint-specific* evidence; no claim of universal optimal forgetting.

### L2 — Recurrent Harness for Agentic Continual Learning (ReHarness)

- **Yuxin Xiao, Zhiyun Lu, Jianyu Wang, Yangtian Zhang, Yang Song, Qifan Wang, Bailin Wang.** Submitted **2026-10-08** (v1); arXiv preprint. [Primary source](https://arxiv.org/abs/2610.12668).
- **Mechanism:** online **Forward Read → trajectory feedback → Backward Revision of retrieved reflections → Forward Write**. Later outcomes can cause *earlier textual memory* to be reconsidered, with attention to overgeneralization or distribution shift.
- **Reported evidence:** three continual-learning benchmarks and three model backbones; comparisons to raw-context, other memory/reflection and prompt-optimization baselines; qualitative examples of corrected experience.
- **Important distinction:** “recurrent” refers to a feedback loop over **textual harness state**, not recurrent Transformer depth, BPTT, or a learned neural hidden state.
- **Limitations / controls:** costs of repeated backward revisions, whether feedback is trustworthy, erroneous revisions, long-run memory drift, versioned rollback, and separate matched-budget ablations for read, revision and write. Check evaluation under novel tasks after correction.
- **Tags:** M2 primary (revision), [M4](../m4-experiential-memory/research.md) secondary (feedback reuse).

### L3 — Useful Memories Become Faulty When Continuously Updated by LLMs

- **Dylan Zhang, Yanshan Lin, Zhengkun Wu, Yihang Sun, Bingxuan Li, Dianqi Li, Hao Peng.** Submitted **2026-05-13**; revised **2026-08-29** (v2); arXiv preprint. [Primary source](https://arxiv.org/abs/2605.12978).
- **Finding:** when LLMs repeatedly rewrite raw experience into consolidated textual lessons, utility may rise and then fall, in some settings **below the no-memory baseline**. The study links regression to the act/schedule of consolidation: the same useful trajectories can lead to different outcomes depending on update timing.
- **Reported stress test:** even using ground-truth example solutions, the authors report **GPT-5.4 failed 54% of previously solved ARC-AGI examples** after consolidation in their experimental protocol. In the ARC-AGI Stream setting, agents that can choose Retain/Delete/Consolidate prefer preserving raw episodes and roughly **double the accuracy** of forced-consolidation counterparts.
- **Interpretation:** strong *counterevidence* to “more frequent consolidation monotonically improves memory.” A quality gate or episodic fallback is a promising **hypothesis**, not a proven general solution.
- **Limitations:** ARC-AGI-style tasks differ from factual long-term assistants, enterprise documents and multiagent operations; results are about tested LLMs, prompting schedules and benchmark setups, not all consolidation algorithms. Raw episodic retention can have unacceptable long-run storage/read costs.
- **Cross-tag:** [M4](../m4-experiential-memory/research.md) for problems transferring experiences into reusable abstractions.

### L4 — MemCut: Every Secret Leaves a Trail in the Long-Term Memory of LLM Agents

- **Cheng Zhang, Xuanning Cao, Zhicheng Zheng, Hui Xiong.** Submitted **2026-10-15** (v1); arXiv preprint and benchmark. [Primary source](https://arxiv.org/abs/2610.21405).
- **Mechanism/evaluation:** benchmark of **456 hand-designed secrets** with annotated *evidence trails* in long conversations. Distinguishes **native recovery** (normal retrieval) from **residual recovery** (reader supplied remaining relevant evidence objects directly). Tests deletion baselines across five memory designs and four LLMs.
- **Reported result:** removing direct disclosures suppresses native recovery more than residual recoverability. LLM-guided deletion of ten additional turns from twenty candidates still permits **39–76% residual recovery** in the stated tests; subset sweeps indicate **candidate coverage** matters.
- **Why it matters:** deleting an obvious statement, hiding it from retrieval or suppressing disclosure via prompt may leave derivable private information in other stored observations/derivatives. Tests must include indirect evidence, lineage coverage and different retrieval routes.
- **Limitations:** recovery depends on benchmark's modeled reader and test design. The numbers are not an audited real-world compliance rate, nor proof that all physical backend copies are removed. Deletion is a policy-and-security requirement, not the same as useful selective forgetting for task performance.

### L5 — AfterDeleteBench: Auditing Post-Deletion Information Persistence in LLM Agents — new high-priority deletion evidence

- **Authors:** Ehsan Saleh, Roozbeh Nahavandi, Sama Salarian, Raef Bassily, Andrew Perrault. **Submitted:** 2026-10-22, v1. **Status:** arXiv preprint / benchmark. **Primary:** https://arxiv.org/abs/2610.31366
- **Test:** a source profile/secret is exposed to a memory system, a derived summary is stored, and the source record is removed. A **matched never-exposed control** has the same harness, background context and query but never saw the target secret. This paired design attributes additional recovery to prior exposure instead of general model knowledge.
- **Reported evidence:** in one Qwen2.5-7B-Instruct hierarchical setup, a direct auditor obtains exposed–control exact recovery gaps of **0.680 on 200 IID profiles** and **0.640 on 200 OOD-generator profiles**, with **zero clean-world exact recoveries** in those conditions. A component-only test, which retains no complete target literal but allows recombination of fragments, still reports substantial recovery. Contrasting flat-memory setups can yield zero recovery under particular test conditions; do not generalize a single observed rate to all stores.
- **Separate endpoints:** (a) initial behavioral disclosure; (b) attempted write-back/re-storage after deletion; (c) recovery in a **fresh session**. Blocking literal re-insertion alone cannot erase prior summaries. A source-lineage closure control can rebuild/delete dependent summaries and suppress recovery *in the specifically instrumented benchmark*, assuming complete lineage.
- **Limitations:** synthetic profile strings, direct/auditor prompts, parser-dependent exact recovery, controlled summarization and model-specific behavior; neither backend backup deletion nor model-weight unlearning was audited. Use **exposed-retained positive controls** to distinguish successful deletion from a weak auditor. Compared with MemCut [L4], the major novelty is **causal paired-world attribution**, not a proof of full semantic erasure.
- **Classification:** M2 primary, security/privacy secondary. The executor is frozen; the changed objects are external source/derived memories.

### L6 — Proactive Memory for Event-Driven LLM Agents (ProactMem) — source-validated memory promotion

- **Authors:** Jinyang Li, Zichuan Fu, Xian Wu, Wenlin Zhang, Yimin Deng, Kaifeng Guo, Guojing Li, Junjia Qi, Xiangyu Zhao. **Submitted:** 2026-10-27, v1. **Status:** arXiv preprint. **Primary:** https://arxiv.org/abs/2610.36967
- **Substrate:** bounded recent events, source-linked temporal trajectories and candidate behavioral regularities. A proposed memory item is checked against underlying events; candidate regularities are also scored by **historical replay** before influencing a proactive intervention. New observations/outcomes are appended after the decision to reduce lookahead.
- **What changes:** entries and their validation/admission state in an **external** memory system; no demonstrated executor-weight updates. Admission of a candidate is distinct from its eventual retention or expiry, which require further tests.
- **Reported evidence:** on author-defined **ProactiveStream** in Sports, Home and Code settings, the paper reports **52.6% aggregate effectiveness F1**, **21.4% relative improvement** versus the strongest tested external baseline. Ablations suggest value from source validation and replay rather than unfiltered abstraction.
- **Limitations:** benchmark and system were built together, event/utility labels and proactive triggers have subjective elements, and joint gains mix interventions with memory validity. Open-ended drift, rare critical facts, changing users, expiry, permission/consent and matched *write+read+replay* latency remain insufficiently established. This does **not** contradict [L3] in general, which studied harmful repeated consolidation in other workloads.
- **Classification:** M2 primary for validation/admission policy, M1 for hierarchical storage and M4 for reuse of behavioral regularities.

### L7 — MoM: Memory of Memory / Provenant Memory (P-Mem) — reversible fact updates

- **Authors:** Bowen Qin, Yao Lu. **Submitted:** 2026-09-07. **Status:** arXiv preprint; included now because it fills a previous omission, not because it was just published. **Primary:** https://arxiv.org/abs/2609.25054
- **State and operators:** typed, source-linked history and a **per-key active frontier**. A candidate can support, supersede, contest, reject, revoke or resolve earlier records. Current facts are committed, but their provenance and displaced versions can remain available for historical queries or reversal.
- **Reported evidence:** the authors report a reader configuration with roughly **4× fewer read tokens** than their strongest compared accuracy baseline; graph-pruned current-state answering reduces stale responses from **19.4% to 10.9%**. Synthetic controlled revision chains show better retraction/rollback than destructive CRUD.
- **Qualifications:** controlled updates often arrive in chronological order. Natural-language key fragmentation and implicit references can cause an apparently “current” materialized view to miss updates; the authors' own results favor read-time pruning in some natural-language settings. Authority disputes, backdated reports, concurrency and storage growth need testing.
- **Deletion boundary:** **superseded ≠ erased**: old sensitive versions persist intentionally. Reversible state is useful for corrections but not automatically compliant with a deletion request. **M2 primary, M1 secondary.**

### L8 — ChronoMem: Version Control and Semantic Rollback for Large Language Model Agent Memory — global history restoration

- **Authors:** Yongye Su, Wujiang Xu, Chaoji Zuo, Elisa Bertino. **Submitted:** 2026-07-30. **Status:** arXiv preprint; previously omitted reference. **Primary:** https://arxiv.org/abs/2607.27773
- **Mechanism:** versioned external-memory writes with immutable events and whole-memory snapshots stored with a current version pointer. A natural-language undo query is matched to earlier snapshots using lexical+dense/hybrid version retrieval; the system **restores active memory state** rather than asking a model to ignore newer facts.
- **Reported evidence:** authors study rollback on adapted LoCoMo and MemoryAgentBench tasks with multiple readers and RAG/prompt/full-history controls; they report positive answer/summarization gains in settings where undo matters. Semantic version Recall@1 is still comparatively low in an example study (**20.5%** for the strengthened selector compared with **12.0%** hybrid baseline).
- **Limitations:** whole-state snapshots cost storage and write time; the reported implementation supports linear histories, not full branching/merging. Rollback cannot undo irreversible external tool actions, guarantee multi-writer consistency, erase historical snapshots/backups, or retroactively remove model-observed information in live prompts. Absolute semantic-target retrieval remains weak.
- **Classification:** M2 primary; M1 secondary for retrieving a historic version. Snapshot restore and physical deletion are different operations.

### L9 — Forgetting Without Restarting: Execution-State Unlearning for Stateful LLM Agents — runtime revocation

- **Authors:** Chao Yao, Yangbo Wei, Zhen Huang, Junhong Qian, Chenle Chen, Shaoqiang Lu, Chen Wu, Lei He. **Submitted:** 2026-09-04. **Status:** arXiv preprint; included as previously missing execution-state boundary. **Primary:** https://arxiv.org/abs/2609.04875
- **Target:** not trained LLM parameters but **dependent execution state**: compressed summaries, plans, prompts, and serving caches generated after an observation that has since been revoked. Removing the source row cannot automatically remove its effect on already-generated downstream state.
- **Proposed mechanism:** provenance graph identifies first dependent step, restores a clean checkpoint prefix and **sanitizes/replays** the suffix without the revoked observation. A controlled deterministic/replayable model provides counterfactual-equivalence arguments; this is not a general guarantee under arbitrary stochastic tools.
- **Reported evidence:** across the authors' tested agent/model configurations, deleting a record may still leave high leakage under direct/suffix probes; selective replay reportedly matches full reset while saving **up to 9× recomputed tokens** in its best tested conditions.
- **Limitations:** correctness requires complete lineage, replayable external observations, controlled randomness, available checkpoints, and safe recovery from tool actions. Irreversible API side effects, shared caches, hidden semantic dependencies and concurrent writers break naïve claims of exact forgetting. This is **not** neural parameter unlearning or an audited privacy-compliance guarantee.
- **Classification:** M2 primary; execution/harness-security boundary secondary. KV-cache matters here only as a *descendant of previously observed memory*, not as general cache research.

## Contradiction / complementarity matrix

| Question | Supportive evidence | Boundary / opposing evidence |
| --- | --- | --- |
| Can selective forgetting improve efficiency? | FadeMem [L1] | Threshold pruning and fusion can lose rare decisive facts; match storage, retrieval and write+read compute |
| Should reflections be revised from new feedback? | ReHarness [L2] | Forced/repeated rewriting can degrade utility [L3]; revision needs independent validity tests |
| Is consolidation necessary to gain from past interactions? | FadeMem uses fusion [L1]; source-validation gates are studied in ProactMem [L6] | Episodic-only controls can match or beat forced consolidation on ARC-AGI Stream [L3] |
| Can historical versions be preserved while current state is corrected? | P-Mem [L7] and ChronoMem [L8] | History intentionally remains available and does not meet a deletion request by itself |
| Can an agent forget information already used in live execution? | Selective replay and counterfactual reconstruction [L9] | Depends on reliable provenance, deterministic/replayable transitions and absence of irreversible side effects |
| Does “not retrieved” imply “forgotten”? | **No.** MemCut [L4] and AfterDeleteBench [L5] show surviving evidence or summaries can yield secrets | Their test/auditor settings do not prove all-world erasure, backend backup policy or model-weight forgetting |

## Next evaluation plan

1. Require a **versioned memory mutation trace**: source IDs, prior/active state, proposed and accepted changes, rollback, derived indexes, timestamp, provenance and separate **valid-at / known-at / ingested-at** semantics.
2. Compare **episodic-only, deterministic summaries, adaptive decay, feedback revision, forced consolidation, validated/gated consolidation and versioned reversible stores** under identical readers/task streams, storage/context and full *write + retrieval + validation/replay + rollback* compute budgets. Include LRU/LFU/TTL and no-memory/full-history/raw-RAG controls.
3. Measure **retention vs stale-memory harm**, including rare but decisive facts, late/backdated corrections, conflicting authority, changing facts and gradual drift. Distinguish rank/strength change, physical entry deletion, derivability from retained summaries, and effect on live plans/cache.
4. Treat **privacy deletion separately**: paired never-exposed and exposed-retained controls, literal/nonliteral recovery, derivation from indirect clues, write-back, fresh-session recall, dependent summaries/indexes and live runtime state [L4,L5,L9]; separately audit backups and access scope. Never claim complete erasure based on one retrieval metric.
5. Test **semantic rollback** with ambiguous version requests, multiple writers, branching/merge and storage amplification [L7,L8]. Distinguish reverting a current view from deleting historical snapshots.
6. Cross-link retrieval substrates to [M1](../m1-memory-systems/research.md), trained lifecycle policies to [M3](../m3-learned-memory/research.md), and experience-to-skill generalization to [M4](../m4-experiential-memory/research.md). No external record edit by itself proves executor-weight updates.

_Review history: 2026-10-20 established L1–L4; 2026-11-03 added L5–L6 and previously omitted L7–L9, and clarified FadeMem's score/threshold/pruning distinction. All results are author-reported preprints; no independent benchmark, deletion audit, execution replay or implementation test was performed. Maintain this file as a corrected living synthesis rather than an append-only scheduler log._
