# Continual & Test-Time Learning Research Watch

_Area established: 2026-10-27. One research watcher; four analytical axes, not four schedulers._

## Research question

How can neural models **adapt to new observations, acquire durable knowledge, and preserve previously learned capabilities** without requiring full retraining after every interaction?

Study continual/lifelong learning and test-time learning together **without assuming they are equivalent**. The central distinction is the lifetime of the adapted state: a test-time memory can improve predictions within one stream while never becoming durable, model-wide knowledge.

## Four complementary axes

| Axis | Primary question | Representative lineage |
| --- | --- | --- |
| **C1 — Classical continual learning** | How to learn sequential tasks/data without catastrophic forgetting or excessive replay? | stability-plasticity, rehearsal/replay, regularization (EWC), expansion, distillation, online CL |
| **C2 — Test-time learning & neural memory** | Can online updates to a neural state/module improve in-sequence memory and adaptation? | test-time training, TTT layers, Titans, MIRAS, ATLAS |
| **C3 — Multi-timescale & self-modifying learning** | Which states/modules should update at which frequencies, and which update rules can themselves be learned? | nested/online optimization, Nested Learning, Hope, continuum memory |
| **C4 — Parametric consolidation & self-adaptation** | How can acquired information transfer from temporary state into durable parameters? | replay/distillation, Sleep, SEAL, continual fine-tuning/self-generated curricula |

Cross-tag related work; classify by the **actual mechanism and persistence guarantee**, not the use of the terms "learning", "memory", "sleep", or "self-improvement". A paper can contribute to several axes; these are research questions, not a single proven chronological progression.

## Operational definitions and exclusions

- **Continual learning (CL):** sequential adaptation to new data/tasks with explicit concern for retaining earlier performance and transferring knowledge. Specify the sequence protocol, whether task identities are known, and whether earlier data can be replayed.
- **Test-time training/learning (TTT/TTL):** adaptation during evaluation/deployment; may update a submodule, weights, state, statistics or representations. An update within one context window **is not evidence** of persistence across sessions/tasks.
- **Test-time adaptation (TTA):** adaptation to test distribution shifts; overlap with TTT is conditional on objectives and state semantics.
- **Neural / fast-weight memory:** learned or gradient-updated parametric state; distinguish fast online state from backbone weights and check reset/restart behavior.
- **Consolidation:** a demonstrated transfer or integration of acquired knowledge into more durable representations/parameters. Do not infer durable incorporation merely from longer context, higher recall, or a saved external artifact.

In scope: original theoretical and empirical work, careful replications, independent negative results, benchmark/evaluation protocols, algorithmic and architectural methods linking sequential updates to long-term retention.

Not sufficient by itself: static RAG, basic KV caching, generic large-context performance, ordinary instruction tuning with no sequential-learning claim, temporary agent notes, marketing demonstrations without mechanism. Track them as cross-references only when relevant.

## Evidence policy

Prefer original papers, proceedings, technical reports, publicly available experiments and implementations; use surveys for the research map. For each significant study, record **title, authors, original date and version, publication status, mechanism, trained/frozen components, adapted state, objective/update rule, state persistence and resets, benchmarks, matched baselines, training/inference cost, failure modes and evidence strength**.

Minimum evaluation questions:

1. **Learning vs retention:** report new-task adaptation alongside old-task retention, forward/backward transfer, and forgetting through *several* updates; consider task-agnostic streams and distribution shift.
2. **Persistence semantics:** do fast weights/optimizer state/auxiliary memory survive sequence boundaries, user sessions, checkpoint reload and consolidation? Which exact tensors change?
3. **Causal attribution:** compare against frozen backbone, plain long-context and retrieval controls when appropriate; compare replay, regularization, ordinary continued training, and matched-size/update-budget alternatives.
4. **Compute and capacity:** account for token/storage budget, extra online gradient steps, offline sleep/replay, synthetic-data generation, model expansion, optimizer state, latency and failure recovery.
5. **Reliability:** conflicting/updating facts, contaminated replay, negative transfer, harmful self-updates, out-of-distribution streams, generalization, and reversibility/rollback.

Distinguish **formal guarantee**, **author-reported controlled experiments**, **preliminary/synthetic evaluation**, **independent replication**, and **speculative architectural interpretation**. Avoid treating a long-context retrieval result as proof of lifelong learning. Do not assume Google contributions are automatically stronger than work from other groups.

## Documentation & update permissions

- [study.md](./study.md): **manually maintained** foundations, surveys, and the reading sequence.
- [research.md](./research.md): **the sole file the scheduled watcher may edit**, as a curated living synthesis, not an append-only run log.
- This README, study guide, root README, other research streams, and their taxonomies are **read-only to the scheduler**.
- Before each research run, read both local reference files and the current research synthesis. Deduplicate, compare against established claims, revise conclusions when supported, and retain compact history/counterevidence. If nothing substantive changed, do not gratuitously rewrite the document.
- If a GitHub write fails or needs approval, **stop immediately and request explicit approval**. Do not repeatedly retry, move to an unauthorized file, or claim the update succeeded.

## Interfaces with adjacent areas

- [Agent Memory M1-M4](../agent-memory/): especially **M3**, for learned memory components and management policies for agents. General test-time neural memory belongs here unless agent-level persistence/use is demonstrated.
- [RNN/SSM](../rnn-ssm/): recurrent and associative-memory architectural mechanics; CL/TTL monitors sequential *learning/updates and retention*.
- [LLM Post-Training](../llm-post-training/): SFT, RL, distillation and optimization; this area tracks **continual capability preservation and adaptive learning**, not generic improvements from post-training.
- [AI Security](../ai-security/): persistent poisoning, unsafe self-modification, contaminated training/replay, and model-update attack surfaces.

## Scheduler response contract

Send a **short report only for meaningful changes**: identify the most important new evidence, its limitations, and link directly to [research.md](./research.md). If nothing important changed, leave research.md unchanged and do not send a notification. The GitHub file is the detailed record.
