# H4 — Executor–Harness Co-Adaptation — Living Research State

_Last reviewed: 2026-10-14 (first focused review)_

**Definition:** change the *task executor model's parameters* together with external harness/program/skill configuration in a **coupled feedback loop**. Merely training a separate **harness evolver** while executor weights are fixed is **H3**, not H4. External-memory edits alone are not weight updates. Read [taxonomy](../taxonomy.md). This is the sole scheduler-writable file for H4; [parent research.md](../research.md) is **human-owned / read only**.

## Executive synthesis

**H4 is no longer only a speculative design space.** Several 2026 preprints study actual adaptation loops involving **executor weights and procedural harness**. But the **degree of co-adaptation varies** dramatically: reusable skills, task-specific workflows, train-time environment/harness curricula and complete executable runtimes are different artifacts.

1. **Strongest directly aligned candidates** include RLHarness (alternating executor RL and procedural-skill edits), Co-Harness (adaptive harness coupled with model RL) and SafeEvolve (joint safety adaptations). These provide experimental support for **restricted H4**, not evidence of autonomous rewriting of an entire production runtime.
2. **Training-time harness optimization vs deployed runtime co-adaptation** should be recorded as an independent axis: some papers optimize a harness used to collect more useful RL rollouts without showing that the resulting inference-time harness keeps evolving in deployment.
3. **Causality is a substantial unsolved problem:** a reported gain from an alternating loop may come mostly from more RL, better trajectory selection or a stronger critic. Strong evidence requires fixed-harness RL, frozen-executor harness search and matched total compute controls.
4. **Safety benefits require special scrutiny:** optimized attack/defense harnesses may improve specific benchmark rates without proving robustness to unseen adversarial strategies.

## Verified primary-source entry points

### A. RLHarness: Co-Evolving Procedural Skills with Reinforcement Learning
- **Primary source:** https://arxiv.org/abs/2609.32326 (September 2026, arXiv preprint).
- **Mechanism:** a *closed alternating cycle* between procedural harness/skill optimization and **RL updates to the executing model policy**. Harness includes procedural instructions, examples and reusable workflows; these are the external artifacts being adapted.
- **H4 fit:** **direct**, but **restricted surface**. Unlike Evo², executor weights update. The work does not establish arbitrary agent loop/runtime self-rewriting.
- **Study questions:** inspect matched RL-only and harness-only controls, variance across seeds, whether skills are used in evaluation or only to collect training trajectories, and if the method transfers to new tasks/models. Separate headline task performance from training/search cost.
- **Evidence status:** research preprint with experimental comparisons, not independently replicated in production.

### B. Co-Harness: Collaborative and Adaptive Agentic Harnesses for RL Training of Large Language Models
- **Primary source:** https://arxiv.org/abs/2609.13739 (September 2026, arXiv preprint).
- **Mechanism:** an *adaptive agentic harness* affects RL data/trajectory generation and changes with the policy-training process. The **executor/policy** is updated using RL, while harness and policy shape subsequent training experience.
- **H4 fit:** **training-loop co-adaptation**, where the harness is a training-side scaffold; do **not** automatically assert that deployed inference-time harness code co-evolves.
- **Study questions:** evidence that the harness itself truly changes (rather than a static RL environment), frozen-harness policy-only baseline, matched samples/compute, transfer beyond train-time tasks.
- **Evidence status:** experimental paper; independent replication pending.

### C. SafeEvolve: Co-Evolving LLM Agents and Safety Harnesses via RL and Red-Teaming
- **Primary source:** https://arxiv.org/abs/2607.22688 (July 2026, arXiv preprint).
- **Mechanism:** agent policy improvement and runtime safety/verification harness revisions alternate or interact using execution/red-team feedback. Unlike purely fixed-executor safety filters, this aims to co-adapt policy and external constraint mechanisms.
- **H4 fit:** **safety-specific co-adaptation candidate**. Verify whether actor/executor model weights are updated for every experimental configuration; reject H4 classification for harness-only ablations.
- **Study questions:** matched attack/test budgets, safety/utility tradeoff, unseen attacks, policy/guard independence, and whether evaluation leaks training red-team prompts.
- **Evidence status:** early evaluation/preprint; safety generalization unproven.

### D. HarnessForge — automated harness development for agentic RL
- **Primary source:** https://arxiv.org/abs/2609.02786 (September 2026, preprint).
- **Mechanism:** a training harness is optimized/forged to support better rollout generation, while the **task model is trained with RL**. This creates a model-training ↔ environment/harness engineering linkage.
- **H4 classification:** **adjacent/conditional**, not strong H4 proof until the paper demonstrates a *coupled changing harness* (not merely an optimized and then frozen training environment) and causal advantage over static-harness RL. Keep distinguished from deploying evolved runtime logic.
- **Evidence status:** relevant to H4 research but requires explicit mechanism-checking.

### E. Skill-R1: Self-Evolution of Model-Skill Co-Adaptation
- **Primary source:** https://arxiv.org/abs/2605.09359 (May 2026, arXiv preprint).
- **Mechanism:** jointly improve executable/reusable procedural skills and the task-solving policy, with training feedback. Skill-level co-adaptation is narrower than a general-purpose runtime.
- **H4 classification:** **restricted procedural-harness / skill category**, provided reported policy update is indeed to the executor; explicitly track model identity in later revisions. Relevant overlap with H3 if a distinct skill-writer is trained instead.
- **Evidence status:** preprint; no claim of fully joint differentiable end-to-end runtime optimization.

## Contrast cases — do not misclassify

- **Evo²** ([arXiv 2610.13025](https://arxiv.org/abs/2610.13025)) trains the **evolver**, and freezes the actual task executor; **H3**, not H4, even if its paper calls this weight–harness co-evolution.
- **DGM, AHE, Meta-Harness, HSI** optimize programs with frozen model weights; **H2**.
- **Standard agentic RL** against a fixed environment/harness updates the executor only and is **not** H4.
- **HarnessBandit** and other dynamic harness-selection policies are not automatically H4: selecting among existing harnesses without a demonstrated changing harness artifact is insufficient.
- Runtime self-reflection, RAG updates, and persistent memories do not constitute parameter training.

## Experimental standards

To support H4 causality, seek a factorial or alternating-control design:

| Experiment | Executor model weights | Harness |
| --- | --- | --- |
| Static baseline | Frozen | Frozen |
| Model-only | Trained | Frozen |
| Harness-only | Frozen | Optimized |
| Co-adapted | Trained | Optimized through feedback |

All comparisons should account for **training examples, optimizer steps, evaluation rollouts, search time, test-time calls, tokens, and leakage**. Test frozen/swapped harnesses and fresh executors to isolate mutual dependence; test over evolving task distributions and on previously solved tasks to measure regressions. Safety work also needs out-of-distribution red-team and adversarial probing.

## Most important unresolved questions

- Does learned **executor–harness specialization** improve *out-of-domain* performance, or only in-distribution RL reward?
- Does improvement survive replacing the harness or executor independently?
- What can safely be automatically modified: reusable skills, prompts, tool schemas, verifier rules, execution loop, permissions or code?
- What is the complexity/compute scaling law for co-adaptation? Can it outperform more RL steps or more frozen-harness search at equal cost?
- How do learned policies become brittle or insecure as the surrounding executable context changes?

## Review history

2026-10-14: first dedicated H4 sweep. The assessment shifted from **"mostly hypothetical"** to **"experimentally investigated in restricted procedural and training harness setups"**. Co-adaptation of **whole production-grade agent runtimes** is not demonstrated by this group of preprints. Claims and classifications should be updated as primary-source detail and independent replication improve.
