# M3 — Learned Memory & Model–Memory Co-Adaptation — Living Research State

_Initial baseline: 2026-10-19. Scheduler-owned. Read [taxonomy.md](../taxonomy.md); parent [research.md](../research.md) is read-only._

## Current synthesis (initial)

1. **Learnable memory can be separate from executor weights.** An encoder, cross-attention reader, latent memory component or memory-control policy may train while the backbone language model remains frozen.
2. **Learned context management must be distinguished from simply prompting a model to edit files.** Document whether the policy is zero-shot, instruction-optimized, or trained via SFT/RL and which parameters receive gradients.
3. **Performance gains require a fair memory/control budget.** Compare identical backbone/data and count training cost, extra memory passes, hidden retrieved context, memory tokens and inference FLOPs; do not interpret a lower token count alone as lower total cost.

## Anchors and preliminary evidence

### A. Memorilla — Adibvafa Fallahpour et al.
- **Primary source:** https://arxiv.org/abs/2610.21375 (2026-10-15; arXiv preprint).
- **Mechanism:** documents are encoded once into reusable embeddings, then a learned query-dependent module generates a fixed number of memory tokens for a frozen LLM.
- **Reported evidence:** **16 memory tokens**, best baseline performance on **five of eight** studied tasks; one TextWorld agent comparison reports success rising from **18.6% full-history** to **24.2%** with learned memory under the paper's setting.
- **Limitations to audit:** task-specific gains/losses, training and encoding cost, length and update of document collection, out-of-domain generalization, memory poisoning and reproducibility.
- **Boundary:** secondary M1 (retrieval substrate); primary M3 because the query-to-memory representation is trained.

### B. Context Language Models (CLM) — Rulin Shao et al.
- **Primary source:** https://arxiv.org/abs/2609.37725 (2026-09-29; arXiv preprint); code: https://github.com/facebookresearch/context-language-models .
- **Mechanism:** maintain context as an updatable file, with the LLM deciding what to retain; the paper studies **zero-shot control, optimized instructions and RL** for context policies.
- **Evidence/limits:** distinguish gains due to learned policy from zero-shot file management and serving optimizations; compare FLOPs, tools, context budget, long-duration workloads and generalization.
- **Boundary:** file editing alone is M2/H1; the learned memory-management policy is M3. Do not imply all reported CLM variants update backbone parameters.

### C. Titans — internal neural memory (boundary case)
- **Primary source:** https://arxiv.org/abs/2501.00663 (2025).
- **Mechanism:** adaptive neural memory during inference / test time, alongside attention.
- **Boundary:** a mechanism for within-stream retention is not automatically durable agent memory; require evidence of persistence/reset semantics and consequential multi-task or multi-session use.

## Monitoring priorities

- Differentiable/latent memory, trainable retrieval, read/write gating, RL memory policies, test-time memory, learned consolidation and genuine executor–memory joint optimization.
- Record frozen and trained components in a parameter-update table. Determine whether evaluation isolates memory learning from merely scaling context or search.
- Track representation capacity, catastrophic interference, temporal corrections, data and sample efficiency, deployment overhead and trust/safety implications.

## Evidence backlog

- Find independent, same-base-model evaluations of learned memory against strong non-learning baselines.
- Establish when trainable memory is preferable to transparent external records in long-horizon production.
