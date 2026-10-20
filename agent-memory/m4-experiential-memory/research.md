# M4 — Experiential & Procedural Memory — Living Research State

_Initial baseline: 2026-10-19. Scheduler-owned. Use [taxonomy.md](../taxonomy.md); parent [research.md](../research.md) is manual/read-only._

## Current synthesis (initial)

1. **A memory of experience is not merely an event log.** Its key test is whether prior trajectories, failures or feedback produce **reusable guidance** for later decisions, preferably on held-out tasks.
2. **Procedural memory need not be parametric.** Reflection notes, skills, executable snippets, checklists or revised action strategies can be stored and consulted without updating executor weights.
3. **Transfer is the key uncertainty.** Apparent continual improvement can come from task repetition, more search/rollouts, cached answers or evaluator leakage rather than generalizable knowledge.

## Anchors and preliminary evidence

### A. Reflexion — Noah Shinn et al.
- **Primary source:** https://arxiv.org/abs/2303.11366 (2023, revised 2023).
- **Mechanism:** converts environmental/task feedback into natural-language reflections stored in an episodic buffer and reused on later attempts, without required weight updates.
- **Evidence/limits:** reported improvements across sequential and code/reasoning benchmarks. Audit reflection reliability, how failures are attributed, token overhead, forgetting, and transfer outside repeated task families.
- **Boundary:** primarily M4; secondary M2 where reflections are maintained over time.

### B. Harness² — Ruiyao Xu et al.
- **Primary source:** https://arxiv.org/abs/2610.14224 (2026-10-09; arXiv preprint).
- **Mechanism:** task-level harness adaptations and a separate loop extracting experience into reusable improvement of future harnessing.
- **Evidence/limits:** the paper reports benchmark gains across several harness/model setups; inspect acceptance criteria, verifier availability, extra trial budget, unseen-task transfer, and whether memory stores general procedures or task-specific patches.
- **Boundary:** primarily [H2](../../agent-systems-engineering/taxonomy.md) for self-evolving harness mechanics, and **M4** for its experience-to-procedure contribution. Do **not** infer updates of executor or evolver weights without direct evidence.

### C. ReHarness (secondary tag: M2)
- **Primary source:** https://arxiv.org/abs/2610.12668 (2026-10-08).
- **Why cross-linked:** repeated feedback changes prior textual reflection. Detailed validity/revision mechanism is owned by [M2](../m2-memory-lifecycle/research.md).

## Monitoring priorities

- Episodic-to-semantic abstraction, reflection, reusable instruction/skill libraries, cross-task feedback, procedural compression, agents acquiring durable habits and skill applicability/expiration.
- Evaluate **held-out transfer, stability–plasticity, retention after task shifts, negative transfer, hallucinated lessons, prompt dependence, per-episode costs and protected tool boundaries**.
- Cross-tag, rather than duplicate, harness evolution, agent training, RAG and ordinary tool-use work when memory is not the primary mechanism.

## Evidence backlog

- Find robust studies isolating reuse of an acquired procedure from extra inference/search.
- Investigate memory-validity tests for generated skills, provenance of learned rules and safe rollback.
