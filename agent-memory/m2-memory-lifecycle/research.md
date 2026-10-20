# M2 — Memory Lifecycle & Evolution — Living Research State

_Initial baseline: 2026-10-19. Scheduler-owned. Read [taxonomy.md](../taxonomy.md); never edit parent [research.md](../research.md)._

## Current synthesis (initial)

1. **Durability without selective updating is a liability.** Relevant memory must be retained while stale, redundant, contradictory, untrusted or low-value entries can be corrected, merged, expired or removed.
2. **Forgetting is an operation on an identified substrate.** A calculated importance/decay score does not directly alter frozen LLM weights: inspect whether the implementation deletes database entries, changes retrieval priority, compacts summaries or removes file segments.
3. **Online revision and forgetting solve different problems.** Decay manages retention under budgets; feedback-guided revision corrects past abstractions when new trajectories contradict them. Their combination remains an open design question.

## Anchors and preliminary evidence

### A. FadeMem — Lei Wei et al.
- **Primary source:** https://arxiv.org/abs/2601.18642 (submitted 2026-01-26; arXiv preprint).
- **Mechanism:** adaptive exponential retention/decay across a two-layer memory architecture, using semantic relevance, access frequency and temporal signals; LLM-assisted conflict handling/fusion.
- **Reported evidence:** evaluations on Multi-Session Chat, LoCoMo and LTI-Bench with reduced storage; paper reports around **45% storage reduction** under its protocol.
- **Limits to audit:** matching memory budget, retrieval context and compute; ablate decay vs consolidation; determine delete-vs-downrank semantics and failure with rare but decisive facts.

### B. ReHarness — Yuxin Xiao et al.
- **Primary source:** https://arxiv.org/abs/2610.12668 (submitted 2026-10-08; arXiv preprint).
- **Mechanism:** forward read and forward write plus **backward revision** of earlier text reflections after observing a subsequent agent trajectory, under an online-stream protocol.
- **Reported evidence:** comparisons over three continual-learning benchmarks and three LLM backbones, including adaptation to distribution shifts.
- **Limits to audit:** revision validity/provenance, extra inference calls, capacity bounds, dependence on accurate future feedback, harmful overwrites and held-out transfer.
- **Boundary:** primarily M2 (record revision), secondarily [M4](../m4-experiential-memory/research.md) (experience-driven adaptation). Textual revisions are not neural recurrent depth or model-weight updates.

## Monitoring priorities

- Memory formation/write selection, deduplication, consolidation, temporal contradiction resolution, versioning/rollback, adaptive forgetting and deletion policies.
- Evaluate stale-memory risk, rare-event retention, privacy deletion, corrupt updates and semantic drift in summaries.
- Compare against fixed-lifetime policies, LRU/LFU or heuristics, simple summaries and full-history under equivalent memory budgets.

## Evidence backlog

- Gather studies with direct lifecycle-op ablations and side-by-side retention/correctness/storage curves.
- Distinguish causal evidence for better forgetting from results caused by a stronger retriever or larger controller.
