# M2 — Memory Lifecycle & Evolution — Living Research State

_Last reviewed: 2026-10-20. First extended literature pass, not just a since-yesterday update. This scheduler owns only this file; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files remain read-only._

## Current synthesis

1. **Forgetting, correction, consolidation and deletion are different operations.** A low retrieval score is not deletion; merging two memories is not proof that both underlying sources remain accessible; rewriting an inaccurate reflection is not a change to frozen model parameters. The storage object and its provenance must be tracked through each operation.
2. **Two positive directions should not be collapsed.** FadeMem reports retrieval/reasoning gains with less storage through adaptive decay and fusion; ReHarness reports benefits from *backward revision* of prior textual reflections as online evidence arrives. Their evidence supports distinct mechanisms and does not establish that their combination is optimal. [L1–L2]
3. **A strong counterexample challenges automatic consolidation.** Zhang et al. find that continuously rewritten memories can become actively harmful, even when generated from useful trajectories; episodic-only retention can remain competitive. This weakens any blanket assumption that consolidation should fire after each event. [L3]
4. **Forgetting on user request requires stronger tests than retrieval disappearance.** MemCut shows that apparent forgetting via the system's normal retriever can leave recoverable traces in remaining memory objects. Source-lineage coverage and *residual* recovery matter. This is a cross-cutting privacy/security issue but the mechanism is directly about the lifecycle of records, so M2 is primary. [L4]

## Lifecycle operation contract — working analytical framework

| Operation | Concrete mutation to audit | Misleading proxy to reject |
| --- | --- | --- |
| Write/select | Adds or rejects a fact/episode plus source provenance | Model saying it will remember |
| Consolidate | Adds or revises abstraction, with links to source events | Shorter prompt or fewer records alone |
| Revise | Edits versioned record, reflecting new feedback/conflict | Appending a new contradictory summary |
| Retain/decay | Changes eligibility/priority or removes entries under policy | Score decay falsely described as hard deletion |
| Forget/delete | Removes or renders inaccessible *all relevant stored descendants* | Failure of normal top-k retrieval alone |
| Expire/temporal correction | Distinguishes valid-at, known-at and supersession | Silently replacing history with a current fact |

This is a proposed engineering/evaluation vocabulary, **not a verified common implementation** of all cited systems.

## Primary evidence

### L1 — FadeMem: Biologically-Inspired Forgetting for Efficient Agent Memory

- **Lei Wei, Xiao Peng, Xu Dong, Niantao Xie, Bin Wang.** Submitted **2026-01-26**, revised **2026-02-06** (v2); arXiv preprint. [Primary source](https://arxiv.org/abs/2601.18642).
- **Mechanism:** dual-layer memory, differential adaptive exponential decay depending on relevance, access frequency and temporal behavior; LLM-guided conflict handling and memory fusion. Decay is applied to external agent-memory retention, **not** backpropagation into the executor weights.
- **Reported evidence:** Multi-Session Chat, LoCoMo and LTI-Bench; the authors report stronger multi-hop retrieval/reasoning with **45% lower memory storage** in their tested regime.
- **What to verify:** exact physical consequences of a decayed score, whether a record is de-indexed, downranked, compressed or erased; sensitivity to rare but decisive memories; fair comparison to LRU/LFU, fixed TTL, summary stores and equal-size RAG. Need per-task ablations and write+read cost.
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

## Contradiction / complementarity matrix

| Question | Supportive evidence | Boundary / opposing evidence |
| --- | --- | --- |
| Can selective forgetting improve efficiency? | FadeMem [L1] | Memory removal may discard rare decisive facts; efficacy depends on workload and matched budgets |
| Should reflections be revised from new feedback? | ReHarness [L2] | Forced/repeated rewriting can degrade utility [L3]; revision needs independent validity tests |
| Is consolidation necessary to gain from past interactions? | FadeMem uses fusion [L1] | Episodic-only controls can match or beat consolidation on ARC-AGI Stream [L3] |
| Does “not retrieved” imply “forgotten”? | **No.** MemCut [L4] separates normal and residual recovery | Its designed secrets/reader protocol do not measure all physical erasure surfaces |

## Next evaluation plan

1. Require a **versioned memory mutation trace**: input source IDs, prior state, proposed update, accepted change, rollback, derived indexes, timestamp, provenance.
2. Compare **episodic-only, deterministic summaries, adaptive decay, feedback revision, forced consolidation and gated consolidation** with the same reader, task streams, storage/context and inference budgets.
3. Measure **retention versus stale-memory harm**, especially after distribution shift, conflicts and reversals of a once-correct fact.
4. Treat **privacy deletion separately**: test residual recoverability, copied/derived records, backups and scope of access; never promise complete erasure based solely on MemCut-style benchmark scores.
5. Cross-link retrieval substrates to [M1](../m1-memory-systems/research.md), trained lifecycle policies to [M3](../m3-learned-memory/research.md), and experience-to-skill generalization to [M4](../m4-experiential-memory/research.md).

_Evidence labels: L1, L2, L3 and L4 are author-reported preprints; no benchmark or implementation was independently reproduced in this review. This replaces/extends the initial scaffold as living synthesis, rather than a run log._
