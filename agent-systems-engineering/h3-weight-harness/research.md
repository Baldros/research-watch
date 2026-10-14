# H3 — Weight–Harness Co-Evolution — Living Research State

_Last reviewed: 2026-10-14 (first focused review)_

**Classification:** repeated harness optimization **also trains the parameterized evolver**, while the agent/task **executor remains frozen**. This is not executor–harness joint training ([H4](../h4-co-adaptation/research.md)). The category asks whether feedback from evaluating harness changes improves the *next harness-editing policy*. See [taxonomy](../taxonomy.md). This file is maintained independently; the human-owned [parent research.md](../research.md) is **read-only**.

## Current synthesis

1. **Evo² is an unusually explicit demonstration of the H3 loop**: an evolver (proposer/reflector) makes harness changes; task execution produces feedback; trajectories update the evolver through SFT and GRPO. Executor weights remain fixed. Therefore do **not** describe Evo² as adapting the executor.
2. **Weight updates should be isolated from larger search budgets.** The strongest H3 evidence is the fixed-executor/fixed-search comparison between trained and untrained evolver. Comparing the *best found harness* alone conflates learning and candidate-search quality.
3. **Co-evolution in this sense is a learning-to-optimize paradigm, not general end-to-end differentiable harness training.** Gradients update evolver weights via training examples/rewards; they do not backpropagate directly through arbitrary Python execution.
4. **Only a small number of clean exemplars are established in this first review.** Skill-proposal policy learning is adjacent, but a paper that updates the **same task executor** from new rollouts should normally be cross-classified H4 or treated as a different mechanism, not imported as H3 uncritically.

## Principal anchor

### Evo²: Evolving the Evolver through Weight-Harness Co-Evolution — Google DeepMind et al.
- **Source:** https://arxiv.org/abs/2610.13025 ; first available October 2026, arXiv preprint (publication status not independently peer-reviewed).
- **Actors:** learned **evolver** `π_φ`, with *proposer* (edit harness) and *reflector* (analyze trials). A separate **executor** `M_θ` carries out downstream tasks with **θ frozen** during harness evolution.
- **Loop:** generate candidate harness program changes → run benchmark tasks → evaluate and reflect → preserve a search tree of harness variants (including cross-branch strategy transfer) → use trajectories to improve **φ** via SFT and GRPO → repeat. This is the EvoTIDE process.
- **Headline comparisons reported in paper:** across tested Qwen setups, Evo² **82.1 vs 62.6** best prior baseline on Polyglot, **48.3 vs 41.0** on SWE-bench Verified, **58.0 vs 56.0** on LiveCodeBench. Differences are task- and protocol-specific; do not compare without matching evaluators/model/search budgets.
- **Isolating the trained evolver:** reported ablation with executor, initial harnesses and search procedure held constant: **Polyglot 61.8 → 78.0; SWE-bench 46.0 → 51.0; LoCoMo 47.2 → 56.5; LongMemEval 57.1 → 67.7**. This directly supports a learned edit policy, but the paper reports **p=0.086 for its SWE-bench ablation**, not strong significance at a conventional 0.05 threshold.
- **Generalization:** authors report transfer of discovered harnesses in **5/6** benchmark transfer conditions without further evolution. Small benchmark universe; verify held-out task splits and candidate selection.
- **Interpretation:** credible proof of concept for *evolver-weight ↔ harness* feedback. Not evidence that an executor learns jointly with its harness; H4 claims require different experiments.
- **Limitations:** concentration in coding and memory tasks; selection bias/best-of-N hazards; expensive evolutionary rollouts and training may be incompletely normalized against H2 search baselines; some proprietary-executor comparisons use different harness selection rules (best of top two for Evo² vs top one for comparator), reducing comparability. Peer review and independent replication outstanding.

## Related work and classification boundaries

- [Darwin Gödel Machine (DGM)](https://arxiv.org/abs/2505.22954), [Agentic Harness Engineering (AHE)](https://arxiv.org/abs/2604.25850), [Meta-Harness](https://arxiv.org/abs/2603.28052) and [Hierarchical Self-Improvement (HSI)](https://arxiv.org/abs/2608.08466) edit agent/harness artifacts with model weights frozen: **H2**, not H3. Even when HSI evolves its *evolver's executable strategy*, this is not *evolver neural-weight training*.
- [Self-Evolution of Model-Skill Co-Adaptation (Skill-R1)](https://arxiv.org/abs/2605.09359) updates skill-generation and model policies under its study design; inspect whether the task executor updates before tagging H3. If executor weights change together with skills, **H4** is more apt. This is an adjacency marker, **not** an additional clean H3 result.
- [EvoHarnessBench](https://arxiv.org/abs/2609.04280) measures harness adaptation and retention; it is not evidence of learning an evolver.
- [RLHarness](https://arxiv.org/abs/2609.32326) has **executor** RL updates plus harness adaptation, thus is **H4** rather than H3.

## Evaluation protocol for future H3 papers

For each work record `φ_before`, `φ_after`, executor `θ`, editable harness surface, reward/evaluator origin and any dataset or rollout reuse. Ask for:
- **Frozen-evolver baseline** with matched edit/search/feedback budgets, same executor, initial harnesses and selection rules.
- Frozen-executor confirmation and separation of train vs final held-out tasks.
- Training-specific ablations (SFT, RL, reflection, crossover), variance across seeds and statistical confidence.
- Gains on *unseen* domains vs exploitation of a particular benchmark's verifier.
- Cost of optimization and training, safety of executing generated harness code, and resilience to evaluator contamination.

## Status / historical notes

2026-10-14: first dedicated H3 synthesis. Evo² treated as principal experimentally evaluated anchor, with a clear caveat that **executor weights are frozen**. No second equally well-substantiated H3 anchor identified in this initial review; absence here is **not proof** no other work exists.
