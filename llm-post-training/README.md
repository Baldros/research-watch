# LLM Post-Training Research Watch

This research stream monitors post-training methods for large language models, with emphasis on techniques that materially change reasoning, tool use, agentic behavior, alignment, efficiency, or training stability after pretraining.

## Scope

Priority topics include:

- supervised fine-tuning, instruction tuning, and high-quality post-training data construction;
- distillation, synthetic-data generation, self-training, and curriculum design;
- reinforcement-learning methods for LLMs, including policy-gradient and group/ranking-based objectives;
- reward models, verifiers, process supervision, outcome supervision, grader agents, and learned evaluators;
- long-horizon and agentic RL, including credit assignment across trajectories, tools, environments, and multi-step tasks;
- exploration, sampling, rejection/filtering, replay, and data-selection strategies used during post-training;
- training stability, variance reduction, clipping/regularization, optimization details, and reward normalization;
- compute scaling and efficiency of post-training pipelines;
- post-training methods for reasoning, coding, math, tool use, and autonomous agents;
- interactions between post-training methods and model architecture when the post-training contribution can be isolated.

## Research objective

The main goal is to understand whether post-training is becoming an increasingly important source of capability improvement in modern LLM systems, and to identify which parts of the pipeline are responsible for those gains.

The research should distinguish:

1. **algorithmic novelty** — a genuinely new or materially modified optimization objective, estimator, sampling strategy, credit-assignment method, or training procedure;
2. **pipeline/system novelty** — new combinations of graders, verifiers, environments, synthetic-data loops, curricula, or rollout infrastructure;
3. **scale effects** — gains primarily explained by more rollout compute, data, model size, or evaluation budget rather than a new method;
4. **model effects** — improvements that cannot be cleanly attributed to post-training because the underlying model or architecture also changed.

When new named methods appear, trace their mathematical and algorithmic lineage where possible rather than treating new terminology as evidence of novelty. Relevant comparisons may include REINFORCE, PPO, GRPO, GAR, preference optimization, verifier-guided training, and related methods, but only when technically appropriate.

## Evidence policy

Prefer primary sources: papers, preprints, technical reports, model reports, training-system disclosures, ablations, and reproducible implementations.

For each important result, inspect whether the claimed gain is supported by controlled comparisons and whether the authors isolate the effect of the post-training method from:

- stronger base models;
- more training or inference compute;
- better or larger datasets;
- different benchmark contamination or filtering;
- improved graders or reward models;
- different rollout environments or harnesses;
- test-time scaling.

Explicitly flag missing ablations, weak baselines, proprietary/unverifiable evaluators, cherry-picked benchmarks, unstable training runs, unreported compute, lack of peer review, and marketing-driven framing.

## Research memory

`research.md` is the living memory for this research stream. Each scheduler run should read the current document before searching and revise it in place rather than append a new dated report.

The document should preserve primary-source links, dates, publication status, the mathematical or algorithmic change, training setup, reward/evaluator design, evidence quality, ablations, compute implications, limitations, and the relation to prior methods. Prior claims and methods should be marked as still relevant, strengthened, weakened, contradicted, superseded, or deprioritized as evidence changes.

Useful historical context should remain visible. Older or superseded methods can be moved into compact archival sections instead of being silently deleted.

The scheduler should normally surface only a selective set of 3–7 genuinely worthwhile new or substantially revised works per run, rather than producing a broad news roundup.
