# M1 — Memory Systems & Retrieval — Living Research State

_Initial baseline: 2026-10-19. Scheduler-owned. Classify with [taxonomy.md](../taxonomy.md). Parent [research.md](../research.md) is read-only._

## Current synthesis (initial)

1. **Memory architecture is an addressability and access problem, not merely a longer prompt.** Different tiers may store transient context, persistent facts, episodic traces and indexes; effective use requires explicit promotion/retrieval policies. How well this transfers across models and tasks remains empirical.
2. **A persistent vector store is only a substrate.** A substantive agent-memory contribution requires coherent read/write semantics and demonstrable benefit to future action, not just a fixed-corpus retrieval benchmark.
3. **Retrieved memory has cost and trust boundaries.** Evaluate stale information, false matches, contradictory entries, overfetch, latency, token budgets and origin/authorization; include no-memory and full-history baselines.

## Anchors and preliminary evidence

### A. MemGPT — Packer et al.
- **Primary source:** https://arxiv.org/abs/2310.08560 (2023, revised 2024; research paper and released research artifacts).
- **Mechanism:** virtual context management inspired by hierarchical memory; data moved between a limited fast context and extended storage with LLM-driven control.
- **Relevance:** canonical baseline for tiered storage and agent-directed retrieval; no implication of updates to executor weights.
- **Evidence/limits:** paper evaluates extended-document analysis and multi-session conversation. Compare on identical backbones and operational budgets before extrapolating to durable multi-agent production.

### B. Memorilla (secondary tag: M3)
- **Primary source:** https://arxiv.org/abs/2610.21375 (submitted 2026-10-15; preprint).
- **Why cross-linked:** it uses reusable document embeddings and query-conditioned access, but its main novelty is a **trained latent memory reader**; primary owner is [M3](../m3-learned-memory/research.md). Do not duplicate detailed evidence here.

## Monitoring priorities

- Concrete agent memory APIs: persistent stores, hierarchies, graphs, episodic traces, long-horizon retrieval and multi-agent sharing.
- What exactly is stored, indexed and loaded into context? What persists after an agent restart or task boundary?
- Compare dense/hybrid/graph retrieval, learned vs unlearned ranking, memory tiering and proactive fetching under matched accuracy/tokens/storage/latency.
- Explicitly measure robustness to obsolete entries, distractors, privacy and cross-user leakage.

## Evidence backlog

- Build a reproducible benchmark matrix (multi-session, long-horizon tasks, mutable facts, retrieval noise).
- Identify studies with stronger matched comparisons against MemGPT-style paging and ordinary RAG.
- Distinguish implementation architecture descriptions from controlled causal evaluation.
