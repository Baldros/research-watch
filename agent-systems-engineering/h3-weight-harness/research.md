# H3 — Weight–Harness Co-Evolution — Living Research State

_Last reviewed: 2026-10-22 (H3 comparative refresh; original publications are earlier)_

**Classification:** repeated harness optimization **also trains the parameterized evolver**, while the agent/task **executor remains frozen**. This is not executor–harness joint training ([H4](../h4-co-adaptation/research.md)). The category asks whether feedback from evaluating harness changes improves the *next harness-editing policy*. See [taxonomy](../taxonomy.md). This file is maintained independently; the human-owned [parent research.md](../research.md) is **read-only**.

## Current synthesis

1. **Evo² is an unusually explicit demonstration of the H3 loop**: an evolver (proposer/reflector) makes harness changes; task execution produces feedback; trajectories update the evolver through SFT and GRPO. Executor weights remain fixed. Therefore do **not** describe Evo² as adapting the executor.
2. **Weight updates should be isolated from larger search budgets.** The strongest H3 evidence is the fixed-executor/fixed-search comparison between trained and untrained evolver. Comparing the *best found harness* alone conflates learning and candidate-search quality.
3. **Co-evolution in this sense is a learning-to-optimize paradigm, not general end-to-end differentiable harness training.** Gradients update evolver weights via training examples/rewards; they do not backpropagate directly through arbitrary Python execution.
4. **The H3 landscape has three different experimentally studied editable surfaces:** complete/partial runtime patches (**Harness-R1**), procedural skill-bank management (**CODESKILL**), and tree-structured harness evolution with learned Proposer–Reflector (**Evo²**). All train the separate *harness optimizer* while the downstream task executor is frozen within the evaluated evolution stage; neither skills nor standalone executor fine-tuning automatically imply H4.

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

## Additional experimentally evaluated H3 lines (verified 2026-10-22)

### Harness-R1: Learning to Edit Executable Runtime Harnesses from Agent Failure Trajectories — Shao et al.

- **Source / status:** [arXiv:2608.02276](https://arxiv.org/abs/2608.02276), submitted 2026-08-03, v1 arXiv preprint; [authors' implementation](https://github.com/DeepExperience/Harness-R1). This predates Evo²; it is an **additional independent H3 anchor**, not a new October publication.
- **Who learns:** a dedicated **Qwen3.5-9B harness engineer** processes batches of failure trajectories and emits validated runtime patches. In each engineer-training/evaluation stage the **target task executor is frozen**. The engineer receives **cold-start full-parameter SFT (877 selected examples) followed by online GRPO**, where rewards come from actual same-batch reruns of that frozen target, not just an LLM judge. Candidate patches are validated and execute as sandboxed hooks around four lifecycle positions. During GRPO, **8 candidate patches per training prompt** are generated.
- **Results:** average task success on **WebShop/ALFWorld/DBBench** improves from **44.3%** (Qwen3.5-9B default) to **53.6%** (Harness-R1), vs **46.4%** for the supervised-only engineer and **48.8%** for the best listed fixed frontier editor. **+9.3 pp** refers to the frozen target's task success, *not a weight change in that target*. In separate sequential training, **direct SFT of the target agent** first yields **59.2%** and a **new target-specific, trained engineer** then reaches **64.2%** (+5.0 pp). This second experiment **does not demonstrate concurrent executor–harness co-training**: actor and corresponding engineer are trained in separate stages.
- **Transfer:** on **20 unseen target models** the learnt editing procedure yields **+7.06 pp average** (each target supplies its own fresh failure traces to create a patch; it is **policy transfer**, not frozen-patch transfer). On **1,270 held-out task instances**, pooled improvement is **+8.9 ± 1.5 pp** across three seeds after selecting patches from ten example failures per benchmark.
- **Costs/limitations:** SFT, online GRPO, and target-SFT experiments each use a node with **8× NVIDIA H800 GPUs**; task reruns are expensive. The study compares fixed editors, but training and search expenditures should be fully normalized. Engineer policies may specialize to target/benchmark; generalization relies on providing fresh target failures and validated sandbox hooks. Three benchmarks and limited seeds do not establish deployment-wide durability. Independent replication/peer review not established here.
- **H3 classification:** **Strong, runtime-patch-focused H3**. It is methodologically distinct from H2 frozen-weight program search and from H4's truly coupled updating of executor and harness.

### CODESKILL: Learning Self-Evolving Skills for Coding Agents — Yanzhou Li et al.

- **Source / status:** [arXiv:2605.25430v2](https://arxiv.org/html/2605.25430) (initial 2026-05-25, **revision 2026-09-25**; arXiv preprint). **Do not conflate** this paper with the unrelated latent-skill paper at arXiv:2609.33243.
- **Who learns:** a separate **Qwen3.5-4B-Instruct skill-management policy** learns to extract, evolve and maintain (add/merge/drop) procedural skills from coding-agent trajectories. **SFT warm-start** (LoRA, 12,856 examples) followed by **GRPO** (500 update steps, 3-stage curriculum) optimizes management; downstream executor **Qwen3.5-35B-A3B stays frozen** during training. Reward mixes **rubric/quality judgments (GPT-5.4-mini)**, alignment evidence and **verifiable improvement in downstream execution over a no-skill baseline**.
- **Performance:** the revised v2 reports macro-average task pass-rate gain **+11.03 pp vs no-skills** and **+5.10 pp vs strongest prompt/memory baseline** over EnvBench, SWE-bench Verified and Terminal-Bench 2. The initial version reported **+9.69 pp**; use versioned comparisons rather than mixing numbers.
- **Evaluation boundary:** the **test-time skill bank is constructed in two preparatory passes over test tasks**. Authors track provenance and exclude a skill if its source includes the *current task instance* during final evaluation. This attempts to prevent self-task contamination, but permits **cross-task transductive information within the test set**. Reported results are therefore **not equivalent to deployment on a wholly unseen distribution with a pre-built bank**; independent replication and a strictly prospective protocol would strengthen the claim.
- **Costs:** the reported training uses **4× H100 80GB GPUs**, ~**20 hours SFT + 210 hours RL**, plus downstream rollouts and a proprietary rubric judge. These costs should be amortized when comparing with fixed prompt-based skill managers. Its editable surface is **skill-bank content/management**, **not arbitrary harness control-flow code**.
- **H3 classification:** **restricted procedural-memory / skills H3**, with a clear interface to external agent-memory studies. Inference-time management and RL-trained manager weights are distinct; **no executor-weight update** is claimed.

### Evo²: clarification from primary-source recheck

The [October 9 Evo² preprint](https://arxiv.org/html/2610.13025) remains a principal H3 demonstration. Its Table 5 holds the **executor, parent harnesses and search procedure** constant while swapping the untrained and learned evolver: **Polyglot 61.8 → 78.0 (+16.3 pp; p=0.001)**; **SWE-bench Verified 46.0 → 51.0 (+5.0 pp; p=0.086)**; **LoCoMo 47.2 → 56.5** and **LongMemEval 57.1 → 67.7** (the last two without the p-values in Table 5). On cross-benchmark transfer, the evolved harness ranks first in **5 of 6** transfer directions.

Its **proprietary-model experiment** (GPT-5-mini / Claude Haiku 4.5) must **not** be cited as an H3 weight-training experiment: *both evolver and executor are frozen* there. The dual-role reflection and crossover search changes make that result primarily **H2-style harness search**. The authors evaluate their better of the **two** top-ranked harnesses there, while baselines report their **single** top-ranked candidate, further limiting direct comparisons.

In the authors' coding setup, the evolved Evo² harness uses **3.6K executor completion tokens and 10.2 LLM calls per held-out task** on LiveCodeBench, versus **6.6–6.9K tokens and 26–37 calls** for HGM/DGM, but **executor inference cost alone excludes harness search + evolver SFT/RL training overhead**.

### Interpretation across H3 methods

| Method | What is edited? | Which neural weights change? | Task executor weights during harness optimization? | Main caveat |
| --- | --- | --- | --- | --- |
| Harness-R1 | Validated runtime/lifecycle patches | Harness engineer (SFT + GRPO) | Frozen within each stage | RL/search/target-specific compute and patch generalization |
| CODESKILL v2 | External procedural skill bank | Skill-management policy (SFT + GRPO) | Frozen | Transductive test-time skill-bank construction; narrow artifact surface |
| Evo² | Executable harness and cross-branch search strategies | Proposer–Reflector evolver (SFT + GRPO) | Frozen | Selection bias, compute matching, significance variable by benchmark |

**What H3 evidence currently supports:** feedback from harness experiments can train a **separate editing/management policy** that improves artifacts around a fixed executor; this is not yet proof of a universally optimal harness or model/harness joint gradient training. Compare **fixed-editor vs trained-editor**, **search+train compute vs fixed search**, **held-out transfer protocol**, **retention/regressions** and **sandbox/permissions** before concluding a method is superior.

**Latest-newness assessment (2026-10-22):** Harness-R1 and CODESKILL are *previously published/revised* papers newly incorporated into this research memory. No new **October 16–22** H3 paper is asserted by this update. Papers with actual **executor weight updates** plus harness adaptation belong primarily to H4, not H3.

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

2026-10-22: integrated pre-existing Harness-R1 (2026-08) and revised CODESKILL v2 (2026-09), clarified Evo²'s statistical significance and frozen-weight proprietary-agent boundary, and specified matched-compute and transductive-evaluation limitations. No additional publication dated October 16–22 was claimed.

2026-10-14: first dedicated H3 synthesis. Evo² treated as principal experimentally evaluated anchor, with a clear caveat that **executor weights are frozen**. No second equally well-substantiated H3 anchor identified in this initial review; absence here is **not proof** no other work exists.
