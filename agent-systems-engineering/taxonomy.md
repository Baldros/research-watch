# Taxonomy of Harness Evolution and Model–Harness Co-Adaptation

_Last updated: 2026-10-14_

This document defines a conceptual taxonomy for research on LLM agent harnesses and their interaction with model training. It complements [research.md](./research.md): the latter maintains evaluated empirical findings, while this file defines concepts, boundaries, and classification rules.

## 1. Definitions and scope

**Model / executor:** a parameterized model `M_θ` that produces decisions, tool requests, or text.

**Harness:** executable software surrounding a model, including prompt/chat-template assembly, tool schemas and invocation, context and memory management, orchestration/agent loops, verification, recovery, permissions, and runtime policy. Changes to a harness need not change neural weights.

**Evolver / optimizer:** a human, algorithm, or learned policy that proposes changes to a harness. Distinguish the *model doing the task* (executor) from the *model optimizing the harness* (evolver): they may or may not share parameters.

**Adaptation:** a change to harness code/configuration, to model parameters, or to both. Merely storing data in external memory or changing inference context is **not** weight adaptation.

The core question is: **what is being optimized, by whom, and does experience from the harness feed back into the weights of the executor or optimizer?**

## 2. Four primary categories

These are conceptual mechanisms, **not a validated historical progression**. A system may combine multiple mechanisms; classification should report the actual optimization loop and the identity of trained models.

| Code | Category | Harness changes? | Optimizer/evolver weights change? | Executor weights change? | Defining evidence |
| --- | --- | --- | --- | --- | --- |
| H1 | **Harness engineering** (manually designed / externally optimized) | Yes | Not required | No | Engineers, fixed algorithms, or searches improve prompts, middleware, tools, workflow or control logic for a fixed executor. |
| H2 | **Self-evolving / automated harness optimization** | Yes, automatically | No in the studied loop | No | An optimizer proposes, executes, evaluates and iterates on harness variants, with fixed model weights. |
| H3 | **Weight–harness co-evolution of an evolver** | Yes, automatically | **Yes** | **No** | Evaluated harness trajectories are used to train the policy that generates or critiques future harness modifications. |
| H4 | **Executor–harness co-adaptation** | Yes | Optional | **Yes** | Executor training and harness adaptation form a coupled, empirically demonstrated feedback loop. |

**Important:** H4 is a research target/classification criterion, not a claim that a specific cited paper has achieved general, fully joint co-training. H3 does not automatically imply H4.

### H1 — Harness engineering

External design and optimization of system components: tool APIs, context compression, verification, execution policies, orchestration, routing, retries, checkpoints, sandboxing and instrumentation. May involve automated hyperparameter search, but does not establish a self-directed evolving harness loop.

Study: robustness under model swaps, hidden costs, regression risks, safety boundaries, latency and accuracy at matched model and budget.

### H2 — Self-evolving harness

A repeated closed-loop search over harness artifacts:

```text
harness H_t -> candidate change -> run fixed executor M -> evaluate
            -> accept/reject/branch -> harness H_(t+1)
```

The model used to propose modifications can be fixed. Search is over external software rather than neural parameters. Differentiate adaptive search from a one-off prompt optimizer.

Study: candidate evaluation budgets, best-of-N selection, held-out generalization, catastrophic harness regressions, and search overhead.

### H3 — Weight–harness co-evolution (evolver-trained)

A loop in which harness experiments produce training examples or rewards that improve the *evolver's parameters*, while an executor can remain frozen:

```text
evolver π_φ -> edits H -> fixed executor M_θ runs tasks -> feedback
       ^                                              |
       |------------ training of φ from trajectories --|
```

**Anchor example:** [Evo²: Evolving the Evolver through Weight-Harness Co-Evolution](https://arxiv.org/abs/2610.13025) (2026). It separates an evolver (proposer/reflector) from a frozen executor; learning to evolve a harness is not the same as updating the executor to use that harness.

Study: controlled fixed-evolver baselines, improvements attributable specifically to parameter updates, distribution shift, learned search policies vs simply increasing search compute, leakage, and transfer of evolved harnesses.

### H4 — Executor–harness co-adaptation

Both `M_θ` (the model executing tasks) and `H` (its external software environment) change as part of a coupled training/adaptation process. A minimal schematic is:

```text
H_t + M_θt -> execution trajectories -> evaluation / training signal
      |                                    |
      +--- adapt H -------------------------+
      +--- update executor parameters θ ----+
```

Possible designs include alternating optimization, joint optimization, and training a model across a distribution of evolving harnesses. Those are **hypotheses/design families**, not evidence that any particular approach works.

Study: whether gains persist with the harness frozen or swapped, causal attribution between weight and harness changes, co-adaptation overfitting, safety and transfer.

## 3. Orthogonal classification axes

The H1–H4 category alone is insufficient. Record these axes for each study:

1. **Optimization target:** prompt/template, tool interface, memory/context, agent loop/graph, middleware, executable harness code, model weights, or several.
2. **Updater identity:** human engineer, deterministic optimizer, frozen LLM, trained evolver, trained executor, joint policy.
3. **Feedback:** static supervised examples, benchmark scores, test outcomes, LLM judge, environment rewards, production telemetry, human feedback.
4. **Training method:** none, search, SFT, preference optimization, RL (state precise algorithm where known), or combinations.
5. **Coupling:** one-way optimization, iterative harness evolution, evolver-weight feedback, executor-weight feedback.
6. **Evaluation:** in-distribution vs held-out tasks; fixed vs swapped models; transfer between benchmarks; sequential adaptation/retention.
7. **Accounting:** compute for search and training, test-time tokens, tool calls, wall-clock latency, monetary costs.
8. **Governance:** executable permissions, policy checks, rollback, provenance, auditability, safe mutation boundaries.

## 4. Decision rules for new papers

1. Identify **executor**, **harness**, **optimizer/evolver**, and any distinct **judge**.
2. Determine exactly **which artifact changes** and **which model weights change**.
3. If the harness is designed/externally optimized but does not self-evolve: **H1**.
4. If it evolves via automated search and all model weights stay fixed: **H2**.
5. If the harness optimizer learns from evolution trajectories while the executor remains fixed: **H3**.
6. If **executor weights and harness are both adapted in a coupled loop**, and that loop is actually evaluated: **H4**.
7. When a paper spans categories, assign **primary + secondary** tags and say why; when evidence is insufficient, use **unclassified / claimed** rather than inferring a stronger category.

Do **not** classify ordinary agent loops, in-context adaptation, tool-use RL with an unchanged harness, or external-memory updates as H3/H4 solely because a model interacts with its environment.

## 5. Evidence standards and failure modes

Compare against **matched budget** alternatives (fixed harness with more inference compute, stronger model, ordinary fine-tuning, non-learning harness search). Ask for independent test sets, multi-seed uncertainty, and proper separation of candidate-selection and final evaluation data.

Common confounders:
- **Best-of-N/search budget** mistaken for learned progress;
- **Evaluator leakage** and benchmark-specific harness overfitting;
- **Model identity ambiguity** (evolver weights updated vs executor weights updated);
- **Unreported training/search costs**;
- **Benchmark gaming** and weak generalization claims;
- **Unsafe automatic code modification**, inadequate permission boundaries or missing rollback.

Evidence labels: *demonstrated with controlled comparisons*, *preliminary/benchmark-specific*, *engineering report*, *proposal/untested*, or *contradicted*. Never treat a conceptual category as proof of a mature research area.

## 6. Interfaces with other research streams

- [Agent Systems Engineering](./README.md): parent stream for system design, orchestration, runtime and production operations.
- [LLM Post-Training](../llm-post-training/): algorithms that update parameters, reward/evaluators, agentic RL; use cross-tags for H3/H4.
- [AI Security](../ai-security/): autonomous code evolution, tool boundaries, permissions, memory contamination and operational attack surfaces.
- [RNN/SSM taxonomy](../rnn-ssm/taxonomy.md): distinct question about *model-internal recurrence* rather than harness optimization.

## 7. Initial research anchors

These are entry points for literature tracking; the current evidence and limitations belong in [research.md](./research.md).

- [Evo²: Evolving the Evolver through Weight-Harness Co-Evolution](https://arxiv.org/abs/2610.13025) — H3 anchor; check exact trained/frozen components and ablations.
- [Beyond Prompts: Measuring and Optimizing LLM Tool-Agent Harnesses](https://arxiv.org/abs/2609.05736) — H1/H2 boundary: explicit harness optimization and reliability analysis.
- [EVOHARNESSBENCH](https://arxiv.org/abs/2609.04280) — evaluates adaptation/retention in harness evolution; benchmark, not itself evidence of H3/H4.
- [Continually improving our agent harness (Cursor)](https://cursor.com/blog/continually-improving-agent-harness) — production H1/ongoing engineering evidence.

## 8. Research-watch operation

Use this taxonomy to tag new evidence, **not** as four independent scheduler instructions. Keep the existing scheduler unchanged until cadence/ownership is reconsidered. `research.md` remains the living evidence synthesis and should capture contradictions, updates, and primary-source references.
