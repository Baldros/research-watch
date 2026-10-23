# M4 — Experiential & Procedural Memory — Living Research State

_Last reviewed: 2026-10-23. New evidence reconciled with the 2026-10-20 baseline; October papers are preprints and their results are author-reported. This scheduler owns only this file; [parent synthesis](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md), and sibling files are read-only._

## Current synthesis

1. **Experience becomes memory only if it changes a later decision.** An inert transcript or log does not demonstrate learning; reusable reflections, policies, guidance or skills must improve behavior on *subsequent* tasks. Reflexion is a foundational text-episodic example; Trace2Skill and Harness² explore more durable procedures. [E1–E3]
2. **Procedural memory spans a spectrum from retrieval to explicit skill artifacts.** Reflexion reuses episode-specific linguistic feedback; MemRL chooses useful episodic experiences by environmental feedback; Trace2Skill condenses multiple trajectories into portable skill instructions; Harness² carries forward experience from harness edits. These mechanisms should not be treated as equivalent. [E1–E4]
3. **A transferable skill needs its own applicability conditions.** Boundary-Aware Skill Memory identifies a *skill imitation trap*: memories of successful trajectories can encourage incorrect tools on superficially similar tasks. More skills need not imply greater accuracy. [E5]
4. **Consolidation can destroy useful experience.** Controlled ARC-AGI experiments show repeatedly rewritten memory banks can become worse than retaining episodes and even worse than not using memory. This is a cross-category counterexample from [M2](../m2-memory-lifecycle/research.md), especially relevant for M4's experience-to-procedure abstraction. [E6]
5. **Procedural memory may itself become the object of training.** APEx trains components of an Executor–Distiller–Planner pipeline. SPEM trains/uses an auxiliary graph-based skill selector alongside external Skill-DAGs. Distinguish such trained controllers from writing external skills with a frozen executor. [E7, E10]
6. **New counterevidence: persistent abstractions can reproduce errors.** Anti-Persistence retains observed trajectories and outcomes while generating and then discarding task-specific guidance. On the authors' held-out tests this is stronger than always persisting distilled rules. This strengthens—but does not universally prove—the warning from E6. [E8]
7. **Memory can cause regressions even with relevant prior experience.** MemDev-Bench explicitly measures rescue and harm in evolving repositories; failures arise when previously successful procedures are reused without reassessing fit. [E9]
8. **Dependency-aware skills and transfer-specific evaluation are important emerging controls.** SPEM uses Skill-DAGs, SEPM uses shared task-dependency graphs and AFTER separates within-task improvement from cross-role and cross-model generalization. Strong observed results remain task- and budget-dependent. [E10–E12]

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

### E8 — More Memory, Worse Agents: Error Reproduction and Anti-Persistence in LLM Agents

- **Ruhan Wang, Kishan Panaganti, Dongruo Zhou.** Submitted **2026-10-19**; arXiv v1 **preprint**. [Primary paper](https://arxiv.org/abs/2610.27406).
- **Mechanism / substrate:** retains original task trajectories and outcomes in **external episodic memory**, but generates task-specific imitation/strategy/correction guidance only when queried and **discards that derived guidance after use**. A feedback-driven selector chooses the mode. The studied memory-update loop does **not** update executor weights.
- **Evidence:** evaluations across **ALFWorld, WebShop and ScienceWorld** with multiple backbones and held-out tasks; the authors report higher mean held-out success and fewer LLM calls versus their studied persistent-abstraction designs. In the paper's targeted discard-versus-persist ablations, discard wins in the reported cases.
- **Novelty relative to E6:** not only shows consolidation can become harmful, but directly tests **persistence of derived rules** as the design variable. Observed “error reproduction” paths are suggestive rather than universally established causal dynamics.
- **Limitations:** three simulated environments, finite update rounds, author-run comparisons, and LLM-call count is **not** total token/FLOP/money cost. Raw experiences can grow indefinitely, and some stable verified skills may rationally be persisted. **M4 primary; M2 secondary** for retention policy.
- **Assessment:** **high-priority alternative baseline** for any proposal claiming value from durable distilled skills.

### E9 — MemDev-Bench: Does Memory Help Code Agents? Benchmarking Memory Systems over Real Repository Histories

- **Guorui Chen, Tao Liu, Zhenzhen Huang et al.** Submitted **2026-10-20**; arXiv v1 **preprint / benchmark**. [Primary paper](https://arxiv.org/abs/2610.27524).
- **Design:** **555 base tasks and 4,869 earlier repository tasks** with relevance-annotated, temporally ordered histories. Compares memory systems with no memory and raw past execution trajectories in code-agent workflows.
- **Finding:** memory sometimes **rescues** failed tasks and sometimes **harms** previously solvable ones—even when histories are related. Wrong transfer occurs when the agent treats old solutions as automatically applicable. Raw historical traces reduce execution steps because they preserve intermediate search/edit/test process information that summaries omit.
- **Evidence caveats:** author-run benchmark; distinguish full task set from smaller end-to-end subsets and keep the same executor/harness. Full traces consume considerably more input tokens, and construction/storage overhead must be included. Its results should not be extrapolated from coding repositories to all agent tasks.
- **Assessment:** adds a valuable **Rescue vs Harm** metric and a strong **full-trajectory baseline**. Supports E5's skill-applicability warning. **M4 primary**, M1/M2 secondary.

### E10 — SPEM: Structure-Aware Procedural Memory for Efficient Skill Use in LLM Agents

- **Ran Zhou, Zehui Kong, Fanyu Lin, Jiaheng Liu, Zhen Zhang, Yuxiang Ren.** Submitted **2026-10-19**; arXiv v1 **preprint**. [Primary paper](https://arxiv.org/abs/2610.26995).
- **Representation:** two stores—**Atomic Skill Memory** and **Skill-DAG Memory** holding ordered, dependency-aware compositions learned from successful executions. The agent reuses high-confidence templates or uses a **Skill-GNN** for state-aware selection of atomic skills.
- **Learning boundary:** distinguish the **trained auxiliary skill-selection component** from the **external, updated DAG artifacts** and the underlying task-executor LLM; changing a DAG does not imply executor-weight updates. **M4 primary / M3 secondary.**
- **Reported evidence:** on **ALFWorld and SkillsBench** the authors report **87.1% ALFWorld success** and **38.38 SkillsBench reward** in a reported configuration, alongside improved runtime/token efficiency against selected baselines.
- **Limitations:** some reuse tests involve closely related or previously seen skills; do not call task-template replay cross-task transfer. Training of Skill-GNN, graph creation, indexing and invalidation after changed tools must enter total cost. Require frozen-selector / flat-library / full-history ablations.

### E11 — SEPM: Self-Evolving Procedural Memory Grounded in Task Graphs for Multi-Agent Collaboration

- **Yi Liu, Junwei Zhao, Li Sun, Jiansong Zhang, Sen Su.** Submitted **2026-10-10**; arXiv **preprint**. [Primary paper](https://arxiv.org/abs/2610.14515); [authors' code](https://github.com/cookieApril/SEPM).
- **Mechanism:** shared **task-dependency graph** plus an evidence blackboard; verified action/dependency paths are distilled into team-reusable procedures, while recurring failures add warnings and constraints. Procedures can be refined, specialized or retired without requiring executor-weight training.
- **Reported evidence:** under **AutoGen and DyLAN**, tested on **ALFWorld, WebArena and OfficeBench**; authors report highest scores against their memory baselines in all six configurations. Under AutoGen, respective gains over no added memory are **+29.85, +25.86 and +13.00 percentage points**.
- **Limitations:** gains mix **coordination/realignment during the current run** with **persisted knowledge reused in later runs**. Need independent cross-task memory-transfer ablations, matched orchestration/search costs, cross-agent provenance and safe rollback. **M4 primary** for reusable procedures; agent-systems-engineering secondary.

### E12 — AFTER: Managing Procedural Memory in LLM Agents: Control, Adaptation, and Evaluation

- **Julia Belikova, Rauf Parchiev, Evgeny Egorov et al.** Submitted **2026-06-22**; arXiv **preprint / benchmark**. [Primary paper](https://arxiv.org/abs/2606.23127).
- **Benchmark:** **382 professional tasks, six roles, 22 procedural skills**, with explicit tests for local refinement, held-out task, cross-role and cross-model transfer.
- **Reported evidence:** one refinement round raises aggregate results by **3.7–6.7 points** depending on the model; diverse multi-model source trajectories yield **73.1% cross-model test accuracy** under the authors' protocol. Some role-specific skills lose effectiveness under transfer.
- **Limitations:** known task/skill annotations simplify selecting the applicable skill; end-to-end deployment must also discover, validate and retrieve procedures. Check overlap across skill/task families, evaluator design and creation-time cost.
- **Assessment:** an important **previously missing transfer-specific evaluation anchor**, not a paper newly published this week. Benchmarks should distinguish *new task, new role, new model* from replay of the original execution.

### Boundary: ReHarness (M2 primary)

[ReHarness](https://arxiv.org/abs/2610.12668) revises earlier textual reflections after new trajectories. It touches M4 via feedback but its principal contribution is correcting past memory, already owned by [M2](../m2-memory-lifecycle/research.md).

## Contradictions and technical distinctions

| Claim | Supporting evidence | Counterevidence / condition |
| --- | --- | --- |
| Reusing stored reflections can improve performance | Reflexion, MemRL | Repeated trials, feedback availability and more inference compute may explain part of improvement |
| Distilled skills transfer between models/tasks | Trace2Skill; APEx in tested regimes | Task-boundary failures and wrong-tool amplification in BASM study; test *dissimilar tool requirements* |
| Consolidating experience is always a good write action | Some distillation/skill results in Trace2Skill and APEx | Zhang et al. show destructive consolidation can severely damage performance |
| External procedures improve without weight updates | Reflexion, Trace2Skill, MemRL | APEx includes trained components; do not describe all procedural-memory research as non-parametric |
| More skill memories are always better | Not supported | BASM [E5], Anti-Persistence [E8] and MemDev [E9] find regressions from unchecked reuse or persistent derived rules |
| Persisting every distilled rule beats reconstructing from episodes | Not established | Anti-Persistence [E8] favors disposable guidance in its tested cases; generality and total cost remain open |
| Same-task skill replay proves task transfer | Not established | SPEM's composition must be separately tested on held-out tasks; AFTER [E12] provides role/model transfer splits |
| Reduced agent steps imply reduced total cost | MemDev [E9] finds raw trajectories shorten runs | Full trajectories increase prompt tokens; include construction, indexing, retrieval and training costs |

## Minimum experiment protocol for an M4 paper

1. Split **source trajectories and target tasks** by time, family/domain and, where possible, model family to reveal memorized answers and shortcut leakage.
2. Include **no-memory**, **raw episodic-only**, **retrieval-only**, **success-only skills**, **skills with boundaries**, and **learned policy** controls as relevant, with equal test-time budgets.
3. Report **skill construction time, total agent calls, skill/token storage, retrieval/tool execution, error recovery and long-run maintenance**, not only downstream pass@1.
4. Inspect negative transfer on tasks **similar in surface form but requiring different tools or policy**.
5. Record **artifact provenance, preconditions, expiration, side effects, human validation, permission scope, and rollback** for executable procedures.
6. Separate **same-task retries, in-family generalization, cross-domain transfer and cross-model transfer**; these are increasingly strong tests and should not be conflated.
7. For learned/distilled artifacts, identify **exact trained vs frozen model components**; M4 memory improvements and H2/H3/H4 harness improvements cannot be casually treated as equivalent.
8. Report **Rescue and Harm separately** with temporally ordered histories, an unchanged executor and frozen memory at evaluation (MemDev [E9]).
9. Compare **raw trajectories, persistent skills, boundary-checked skills and disposable query-time guidance** at equal *total* write + read + execution budgets (E5, E8, E9).
10. For compositional skill selectors such as SPEM, ablate **graph/skill storage separately from the trained GNN controller**; report held-out template transfer.

## Next priorities

- Reproduce a small same-backbone comparison: **Reflexion-style episodic notes → Trace2Skill-style consolidated procedural instructions → BASM-style boundary-aware skills**, with forced-consolidation and raw-trajectory baselines.
- Test whether preserving provenance and failure conditions reduces negative transfer while containing latency and token growth.
- Measure procedural **staleness** after tools/APIs change: is a retrieved skill valid, should it be retired, or can it be repaired safely?
- Monitor independent replications and preprint revisions; distinguish meaningful new evidence from reuse of branded “self-evolution” terminology.
- **New priority:** compare raw episodic traces, permanent skill libraries, boundary-aware skills and *on-demand temporary* guidance on identical held-out tasks, recording Rescue/Harm, cross-role transfer, latency, tokens and artifact-construction costs.
- **New priority:** evaluate dependency-aware Skill-DAGs on task/tool drift; avoid treating replay of the task that created the DAG as evidence of transfer.

_All numeric gains are author-reported and tied to specific experimental protocols; most new October works are unreviewed arXiv preprints. No benchmark or code was independently rerun. This is a living synthesis of E1–E12, not a scheduler event log._
