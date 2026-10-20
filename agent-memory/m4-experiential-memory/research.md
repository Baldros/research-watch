# M4 — Experiential & Procedural Memory — Living Research State

_Last reviewed: 2026-10-20. First extended evidence pass. This scheduler owns only this file; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files are read-only._

## Current synthesis

1. **Experience becomes memory only if it changes a later decision.** An inert transcript or log does not demonstrate learning; reusable reflections, policies, guidance or skills must improve behavior on *subsequent* tasks. Reflexion is a foundational text-episodic example; Trace2Skill and Harness² explore more durable procedures. [E1–E3]
2. **Procedural memory spans a spectrum from retrieval to explicit skill artifacts.** Reflexion reuses episode-specific linguistic feedback; MemRL chooses useful episodic experiences by environmental feedback; Trace2Skill condenses multiple trajectories into portable skill instructions; Harness² carries forward experience from harness edits. These mechanisms should not be treated as equivalent. [E1–E4]
3. **A transferable skill needs its own applicability conditions.** Boundary-Aware Skill Memory identifies a *skill imitation trap*: memories of successful trajectories can encourage incorrect tools on superficially similar tasks. More skills need not imply greater accuracy. [E5]
4. **Consolidation can destroy useful experience.** Controlled ARC-AGI experiments show repeatedly rewritten memory banks can become worse than retaining episodes and even worse than not using memory. This is a cross-category counterexample from [M2](../m2-memory-lifecycle/research.md), especially relevant for M4's experience-to-procedure abstraction. [E6]
5. **Procedural memory may itself become the object of training.** APEx alternates learning of Executor/Distiller/Planner and also performs skill-guided test-time adaptation; it crosses into [M3](../m3-learned-memory/research.md). Carefully name what is trained and avoid mistaking an external skill for a model-weight update. [E7]

## Research boundaries

**Episodic memory** retains individual runs/observations. **Reflections** interpret runs and record lessons. **Procedural memory** stores strategies, policies, scripts, instructions or executable skills; it should carry applicability, risk and recovery conditions. **Skill learning** can mean either (a) writing external reusable artifacts with frozen model weights, or (b) optimizing a trained controller using those artifacts; classify the mechanism actually tested.

**Primary M4** when the contribution is *abstracting or reusing experience for future behavior*. **Primary M2** when the contribution is revision, consistency and forgetting of existing records; **primary harness H2** when the contribution is automation of harness modification (cross-tag procedural learning here if significant).

## Primary evidence

### E1 — Reflexion: Language Agents with Verbal Reinforcement Learning

- **Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao.** Submitted **2023-03-20**, revised **2023-10-10**; foundational research preprint. [Primary source](https://arxiv.org/abs/2303.11366).
- **Mechanism:** after task feedback, the LLM writes natural-language reflections into episodic memory; later task attempts read these reflections. “Verbal RL” in the title does **not** require policy-gradient updates to the executor's weights.
- **Author-reported results:** sequential decision-making, coding and language reasoning. The headline HumanEval pass@1 **91% versus GPT-4 80%** is an original author-reported comparison, **not** a fair same-backbone proof that memory alone caused the gap.
- **Limits:** repeated retries on the same/equivalent tasks can inflate gains; reflections can encode mistaken causal hypotheses; inspect task feedback quality, rollout/token cost and genuine transfer.
- **Interpretation:** archetypal **episodic feedback → textual guidance**. Secondary M2 only if revisions are studied.

### E2 — Trace2Skill: Distill Trajectory-Local Lessons into Transferable Agent Skills

- **Jingwei Ni et al.** Submitted **2026-03-26**, revised **2026-06-04** (v5); **work in progress / preprint**. [Primary source](https://arxiv.org/abs/2603.25158).
- **Mechanism:** parallel, inductive consolidation of many execution trajectories into a versionable skill directory. Aims to distill failure patterns and workarounds into reusable standard operating procedures (SoPs), not merely retrieve the nearest complete trace.
- **Reported evidence:** tests across office, mathematics and visual QA tasks; authors report transfer across model scales/families and to some OOD settings. The headline up-to **+57.65 percentage points** on WikiTableQuestions concerns a particular transfer setup (skills from Qwen3.5-35B trajectories used by Qwen3.5-122B), **not** a universal expected gain.
- **Limits:** potential skill overfitting, cost and quality of trajectory pool, contamination between source tasks and destination tasks, runtime cost of loading/retrieving artifacts, external execution/permission risks, and incremental skill-update regressions.
- **Interpretation:** strong candidate for explicit external procedural memory; test independently with fixed executor and held-out tasks.

### E3 — Harness²: Recursive Agent Harnessing for an Open World

- **Ruiyao Xu et al.** Submitted **2026-10-09**; arXiv v1 preprint. [Primary source](https://arxiv.org/abs/2610.14224).
- **Mechanism:** **task-level recursion** proposes/probes/composes harness adaptations; **harnessing-level recursion** extracts experience from such edits to improve adaptation to later tasks. Novelty relevant to M4 is **experience gained from harness modification becoming reusable future guidance**, not merely that the current task is optimized.
- **Reported evidence:** three professional benchmarks, four model-harness settings; author-reported gains of **up to +11.5 points WorkBuddy-Bench, +4.5 JobBench, +11.8 percentage points LAB** for tested conditions, with added improvements from more recursion budget.
- **Limits:** attribution to retained experience versus more test-time search/probes; absence of oracle on open-world tasks, held-out transfer, persistence format, rollback and permission safety. **Do not infer model-weight co-training** from “recursive learning.”
- **Classification:** [Harness H2](../../agent-systems-engineering/taxonomy.md) primary for harness self-evolution; M4 only for experience-to-improved-harnessing contribution. Avoid duplicating the H2 experimental ledger here.

### E4 — MemRL: Self-Evolving Agents via Runtime Reinforcement Learning on Episodic Memory

- **Shengtao Zhang et al.** Submitted **2026-01-06**, revised **2026-02-12** (v2); arXiv preprint. [Primary source](https://arxiv.org/abs/2601.03192).
- **Mechanism:** a **non-parametric feedback-driven** episodic memory system with two-phase retrieval that selects high-utility strategies rather than merely semantically similar ones. Despite “RL” in the title, reported runtime evolution does **not** require updating executor weights.
- **Reported evidence:** HLE, BigCodeBench, ALFWorld and Lifelong Agent Bench; authors report improvements over their memory/retrieval baselines.
- **Limits:** identify how feedback is scored and stored, whether benchmark tasks repeated, retrieval selection overhead, evaluator/gold-answer leakage, long-run strategy degeneration and held-out improvements. Contrast against simple retrieval with equal reranking compute.
- **Classification:** M4 primary; M1 retrieval mechanism secondary. Distinct from **Memory-R1** (trained policy; [M3](../m3-learned-memory/research.md)).

### E5 — When Not to Imitate: Boundary-Aware Skill Memory for Reliable Tool-Use LLM Agents

- **Zihan Lin et al.** Submitted **2026-08-23**, **accepted EMNLP 2026 Findings** according to arXiv. [Primary source](https://arxiv.org/abs/2608.22339).
- **Finding:** success-only skill distillation can raise agent confidence in a *wrong* tool choice when a new task superficially resembles old successes. The paper reports a **47% larger wrong-tool margin** versus a memory-free control under its probe.
- **Mechanism:** **BASM** enriches each skill with explicit **applicability conditions, risk cues, avoidance rules and recovery notes**; agent can select, suppress or repair a procedure based on current state.
- **Reported results:** gains **up to 23.8% in AppWorld success, up to 5.0% BFCL accuracy** versus success-distilled skill baselines, and **4.6% lower AgentDojo attack success rate**, under specific tested configurations. These maxima are not pooled averages.
- **Interpretation:** strong counterevidence to unconditional imitation and a concrete engineering requirement for procedural memory: every reusable recipe should declare its boundary.
- **Limits:** measured tool-use datasets and tested backbones may not generalize to all skills; condition extraction can itself be wrong, and recovery paths may introduce overhead.

### E6 — Useful Memories Become Faulty When Continuously Updated by LLMs (cross-category counterevidence)

- **Dylan Zhang et al.** Submitted **2026-05-13**, revised **2026-08-29**; arXiv v2 preprint. [Primary source](https://arxiv.org/abs/2605.12978).
- **Main result:** forced repeated experience consolidation can corrupt previously useful lessons; memory utility first improves and may later dip below *no-memory*. Their ARC-AGI Stream episodic-retention condition can be competitive with or better than forced consolidation; one experiment reports **54% previously solved ARC-AGI cases** failing following consolidation with GPT-5.4.
- **Why M4 cares:** “generalize experience into a skill” is an imperfect lossy inference, not guaranteed safe progress. Preserve source traces, version procedures, compare episodic-only against always-consolidate and gate reuse/overwrite.
- **Boundary:** detailed lifecycle degradation belongs in [M2](../m2-memory-lifecycle/research.md); this is a warning for generalization/abstraction.

### E7 — APEx: Distillation of Agent Procedural Experience for Adaptive Deep Research Question Answering

- **Jie Ding, Rui Sun, Xinyuan Zhang, Zeyu Zhang, Xin Liu.** Submitted **2026-09-02**; arXiv preprint. [Primary source](https://arxiv.org/abs/2609.02253).
- **Mechanism:** stores *instance-level execution trajectories* alongside *category-level procedural skills*. A coupled **Executor–Distiller–Planner** pipeline uses alternating GRPO stages for distillation and adaptation; at inference, skill priors guide **test-time RL** of the planner.
- **Reported evidence:** seven benchmarks, author-reported **+14.7 points versus GPT-5.4** and **+3.0 points against the strongest memory-augmented comparator** in the claimed evaluation.
- **Limits:** extremely important to isolate contribution of learned skill library from the training, planner updates, increased test-time optimization and possible backbone differences. Need freeze-policy/freeze-skill ablations and equal test-time compute; do not attribute all improvements to “memory.”
- **Classification:** M4 primary for experience-to-procedure research, **M3 secondary for trained policy**, potentially post-training/harness cross-tag. The name “self-improvement” does not imply all weights are frozen.

### Secondary cross-link — ReHarness (M2 primary)

[ReHarness](https://arxiv.org/abs/2610.12668) revises earlier reflections after observing new task trajectories. It touches M4 through experience feedback, but its principal novelty is backward correction of existing memory and is maintained under [M2](../m2-memory-lifecycle/research.md).

## Contradictions and technical distinctions

| Claim | Supporting evidence | Counterevidence / condition |
| --- | --- | --- |
| Reusing stored reflections can improve performance | Reflexion, MemRL | Repeated trials, feedback availability and more inference compute may explain part of improvement |
| Distilled skills transfer between models/tasks | Trace2Skill; APEx in tested regimes | Task-boundary failures and wrong-tool amplification in BASM study; test *dissimilar tool requirements* |
| Consolidating experience is always a good write action | Some distillation/skill results in Trace2Skill and APEx | Zhang et al. show destructive consolidation can severely damage performance |
| External procedures improve without weight updates | Reflexion, Trace2Skill, MemRL | APEx includes trained components; do not describe all procedural-memory research as non-parametric |
| More skill memories are always better | Not supported | BASM's imitation-trap experiment finds wrong-tool confidence rises in some retrieval-heavy settings |

## Minimum experiment protocol for an M4 paper

1. Split **source trajectories and target tasks** by time, family/domain and, where possible, model family to reveal memorized answers and shortcut leakage.
2. Include **no-memory**, **raw episodic-only**, **retrieval-only**, **success-only skills**, **skills with boundaries**, and **learned policy** controls as relevant, with equal test-time budgets.
3. Report **skill construction time, total agent calls, skill/token storage, retrieval/tool execution, error recovery and long-run maintenance**, not only downstream pass@1.
4. Inspect negative transfer on tasks **similar in surface form but requiring different tools or policy**.
5. Record **artifact provenance, preconditions, expiration, side effects, human validation, permission scope, and rollback** for executable procedures.
6. Separate **same-task retries, in-family generalization, cross-domain transfer and cross-model transfer**; these are increasingly strong tests and should not be conflated.
7. For learned/distilled artifacts, identify **exact trained vs frozen model components**; M4 memory improvements and H2/H3/H4 harness improvements cannot be casually treated as equivalent.

## Next priorities

- Reproduce a small same-backbone comparison: **Reflexion-style episodic notes → Trace2Skill-style consolidated procedural instructions → BASM-style boundary-aware skills**, with forced-consolidation and raw-trajectory baselines.
- Test whether preserving provenance and failure conditions reduces negative transfer while containing latency and token growth.
- Measure procedural **staleness** after tools/APIs change: is a retrieved skill valid, should it be retired, or can it be repaired safely?
- Monitor independent replications and preprint revisions; distinguish meaningful new evidence from reuse of branded “self-evolution” terminology.

_All numeric gains above are author-reported and tied to specific benchmark settings. No benchmark or code was independently rerun during this initial review. This document is living synthesis, not a chronological scheduler log._
