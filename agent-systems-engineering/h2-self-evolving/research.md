# H2 — Self-Evolving Harness — Living Research State

_Last reviewed: 2026-10-27 (matched-budget and continual-retention evidence)_

**Category:** a system iteratively rewrites externally stored harness artifacts based on execution feedback while **executor and optimizer/evolver neural weights remain fixed**. A meta-evolver that modifies the *evolver's source code* but does not train its weights remains H2. Exclude evolver-weight learning ([H3](../h3-weight-harness/research.md)) and executor-weight co-adaptation ([H4](../h4-co-adaptation/research.md)). See [taxonomy](../taxonomy.md). This is the scheduler's **only** writable research file; [parent research.md](../research.md) is human-owned/read-only.

## Current synthesis

1. **Closed-loop harness search can improve a fixed model, but is not a guaranteed generalizable gain.** AHE, DGM, Meta-Harness and new SSR provide positive controlled examples. *Rethinking Evaluation* reports **no average held-out transfer gain (63.2% vs 63.2%)** and task-level sampling can beat harness search at comparable feedback budgets.
2. **Search design matters more than raw experience count:** SSR tests representative experience ranking, multiscale edits and shared-batch candidate racing. A 60% reduction in its *candidate validation executions* is **not** a 60% reduction in total cost.
3. **Harness evolution is non-monotonic:** EvoHarnessBench, *Continual Learners* and robot-harness experiments reveal forgetting, interference and noisy candidate promotion. Strict gates reduce some regressions but cannot prove global retention under reused validators.
4. **Self-judgment, benchmark selection and search budget remain confounders.** RHO/UCoE's judge signals can be useful while still amplifying correlated errors. Always compare against direct task attempts and require fresh held-out tasks.
5. **Safety and functional correctness need separate tests:** Harmony's Pareto search and reported in-sample grader exploitation show why apparent reward improvements cannot certify security. Protect mutation boundaries with sandboxing, permissions, provenance and rollback.
6. **Boundary matters:** external memory and executable code are different H2 surfaces; HERA co-evolves task environments without weight training; Evo² learns a neural evolver and therefore belongs to H3; robot-policy distillation updates executor weights and is not a pure H2 result.
7. **Evidence standards strengthened:** track fixed model/optimizer, actual artifact delta, sampling and selection budgets, performance on previously passed tasks, cross-model and OOD transfer, independent success/safety oracles, and total cost per successful rollout.

## Recent/newly identified (October 2026)

### A. Verifier–Harness Co-Evolution for Training-Free Self-Improvement on Open-Ended Tasks (UCoE)
- **Authors / date / status:** Tianyi Xu, Yaolun Zhang, Yiran Wu, Joe Nguyen, Shengyu Dai, Saba Moeinizade, Zhenwen Shao, Qingyun Wu, Huazheng Wang; **2026-10-13**, arXiv preprint.
- **Source:** https://arxiv.org/abs/2610.17609 ; code reported at https://github.com/tianyi0216/Unified-Co-Evolution .
- **Mechanism:** all actor and verifier-model weights **frozen**. UCoE generates and selects *executable verifier programs*, selected on contrast families and worst-case discrimination ("driving capacity"); the verifier diagnoses a draft, controls one revision and acceptance, and gates a **bounded procedural-memory update**. Subsequent accepted repairs change future harness state; verifier code also evolves. This is **verifier-program ↔ harness-state co-evolution, not H3 or H4**.
- **Evidence:** against matched direct generation on six open-ended benchmarks, reported **59.00% and 57.19%** tie-adjusted win rates for Qwen3-8B and Qwen3.8-27B. On three verifiable benchmarks, task accuracy increases **36.06% → 38.48%**. The coding-agent harness demonstration on **75 SpreadsheetBench-2 tasks** reports exact match **17.3% → 22.7%**; small-sample transfer probe, not proof of broad deployment benefit.
- **Limitations:** open-ended scoring is itself model-judged; contrast families may miss real errors; one mistaken verifier decision can contaminate persistent harness state. Extra semantic evaluations raise cost. Outcome/preference gains are sensitive to evaluators and benchmark setup.

### B. Harness Self-Evolution: Theory, Algorithms, Evaluation, Applications, and Impact (survey/working draft)
- **Authors / date / status:** Chuyang Wei, Haoxiang Guan, Minghang Zhu, Kexin Zhou, Jiyan He, Yifei Shen; **2026-10-12 working draft**, not independently peer-reviewed on this review.
- **Sources:** https://zgcagi.github.io/Harness-RSI/ ; companion reading list https://github.com/zgcagi/Harness-RSI .
- **Contribution:** surveys agent improvements via instructions, external memory, workflows, tools and runtime modification **without weight changes**; explicitly separates selection, activation, attribution and improvement evidence.
- **Audit reported by authors:** 95 methods included; 93 have a primary comparison; **zero** of their 93 scored methods meet *all four* proposed evidence criteria (26 none, 27 one, 33 two, 7 three). Particularly few meet their activation and attribution standards. **These are rubric-dependent audit results, not an objective ranking of each method's validity**; the corpus stops at **2026-10-08**, so it omits UCoE and future work. This is a valuable checklist for future watchers.

### Successive Scale Racing (SSR): Forging More Tasks into Better Harnesses — Jingzhe Liu et al.

- **Primary:** https://arxiv.org/abs/2610.28815 (2026-10-20, preprint). **Most important new H2 paper from this round.**
- **Loop:** executor and proposer weights **frozen**. Representative Evidence Ranking (RER) chooses high-quality, nonredundant failure records; a fixed-weight proposer makes harness edits from nested 25/50/75/100% evidence subsets; **Successive Scale Racing** tests alternative executable harnesses on progressively larger shared batches, rejecting weak candidates and promoting a strict empirical winner. Optimizes prompts, scripts and control artifacts, not weights. Note: a series of challenger candidates is not an unrestricted archive of all surviving variants.
- **Evaluation:** SWE-bench Pro Verified (80 train/20 selection/100 held-out), DeepSWE (50/13/50) and GAIA-2 (80/20/100), on Codex/GPT-5.5 and Claude Code/Opus 4.8; the headline averages use three evolution runs and three final test rollouts.
- **Results:** Codex SSR reports **65.7/68.0/38.0%**, vs strongest prior comparator **62.7/66.7/36.3%** on those benchmarks. Claude Code reports **63.3/65.3/36.0%**, vs **62.0/64.0/34.3%**. Differences of **1.3–3.0 percentage points** require caution: limited seeds, no robust multi-comparison significance claim. In an additional 12-candidate selection audit, racing achieved **65.3% vs 65.7%** for exhaustive validation, while using **40% of validation executions**. This **does not** mean evolution as a whole costs 60% less: candidate generation and training-task rollouts still count.
- **Ablations/budget:** evidence scaling 40→200 tasks also **increases** execution allowance, so is not a fixed-compute scaling result; equal-cap comparisons support representative evidence and multiscale generation, but smaller than headline gains. Racing guarantees apply to a bounded validation pool/round, not prospective deployment under adaptive reuse.
- **Threats:** reuse of some training trajectories in selection, benchmark-specific edit preferences, costly search and unsandboxed mutable code risks; need held-out domain transfer, tool/version drift and uncertainty estimates before describing broad autonomous improvement.

### Are Self-Evolving Agents Continual Learners? — Xiaoyin Chen et al.

- **Primary:** https://arxiv.org/abs/2610.25858 (2026-10-19, preprint). **Important counterevidence on retention**.
- **Setup:** one frozen Gemini 3.6 Flash model, four ordered task-distribution streams (BIRD, BFCL, ScienceWorld and mixtures; roughly 100–200 tasks). Compares **ReasoningBank** (retrieved external experiences; boundary with memory management), **GEPA** (global strategy/prompt rewrites) and **Meta-Harness** (prompt, skills, memory and executable harness.py edits, validated before admission).
- **Results:** across streams, GEPA ends at **+3.7 pp** while exhibiting **plasticity +6.2** and **stability −4.3 pp**; ReasoningBank **+2.5** / **+3.3** / **−1.9**; Meta-Harness **+1.4** / **+2.2** / **−2.1**. These are stream/decomposition statistics, not absolute success rates. A topic-drift GEPA case initially gains +7.5 points, then loses 9.5 to finish at −2.0.
- **Mechanisms:** global prompt updates interfere with unrelated skills or overwrite earlier instructions; memory retrieval can collide when similar-looking requests require *opposite* workflows. Stricter validation gates can reject harmful candidates but also reduce adaptation.
- **Limits:** single frozen model, selected task streams, relatively short trajectories; insufficient to rank long-run general-purpose memory architectures. **Implication:** measure **the complete old-versus-new task performance matrix**, not just average score on the newest batch.

### Rethinking the Evaluation of Harness Evolution for Agents — Yike Wang et al.

- **Primary:** https://arxiv.org/abs/2607.12227 (2026-07-14; updated 2026-10-05, v4). **Earlier work added for counterevidence**, not a fresh October 20 publication.
- **Mechanism and controlled question:** compared fixed-weight, batch/per-task harness evolution against **parallel sampling and sequential refinement** of actual task solutions on Terminal-Bench 2.1, with matching K=5 feedback/inference attempts on three frozen frontier models (Opus 4.6, GPT-5.4, GPT-5.4 mini). Direct task-level sampling is **not** H2 but is a necessary budget-matched control.
- **Negative evidence:** without unit tests, **72.3% parallel sampling** vs **67.4% harness evolution** pass@1; with unit tests/oracle selection, **82.0%** vs **69.7%** pass@1. On **34 disjoint held-out Terminal-Bench tasks** (45 training/10 selection), evolved-vs-static success is **64.5 vs 63.3** (Opus), **72.1 vs 72.1** (GPT-5.4), **53.0 vs 54.4** (mini): **63.2% average in both groups**. Distinguish pass@1 from oracle-selected pass@5.
- **Positive boundary:** task-specific evolution shows stronger gains in long-horizon ARC-AGI-3 / EdgeBench games with granular environmental feedback. Reported **RHAE efficiency scores**, e.g. **77.5 vs 44.3** in one setup, are **not success percentages**, nor evidence of reusable harness transfer.
- **Limits:** benchmark/version/model specificity; estimates may exclude harness-edit overhead; no broad production test. **Interpretation:** H2 helps when experience yields *reusable structural bottleneck fixes*, not automatically when feedback could instead fund more direct attempts.

### Harmony Harness: Multi-Objective Self-Evolution of Agent Harnesses — Weichen Yu et al.

- **Primary:** https://arxiv.org/abs/2610.15150 (2026-10-10, preprint); first introduced before this review.
- **Loop:** frozen Gemini 3.6 Flash proposer/executor; executable, rollbackable candidate edits to tools/skills/control logic, with an evolving Pareto population over **functional correctness AND exploit resistance**. Decomposition distinguishes portable vs environment-specific artifacts; dynamic task-time composition picks components. **H2**, not evolver neural-weight training.
- **Results:** on held-out SecRepoBench, **secure-pass@1 (functional AND secure)** rises **50.0→86.7% in-domain** and **36.7→66.7% OOD**. On BaxBench, reported mean **61.3%** vs **42.9%** baseline. The reported multiobjective ablation favors decomposition-based search, but task samples are small (e.g. 30/30/30 SecRepoBench; 28/28/28 BaxBench).
- **Counterevidence within the paper:** an alternate admission/validation criterion obtains **perfect in-sample scores through grader exploitation**. Thus a high in-sample 'secure' reward can indicate **reward hacking**, not real safety. Some OOD transfer evaluations use related benchmark families; compute and generated-code sandboxing need fuller accounting.
- **Implication:** carry safety as its own objective with independently executable, held-out attack tests; don't infer safety from task completion.

### HERA: Harness–Environment Co-Evolution for Reliable Agentic Abstention — Han Luo et al.

- **Primary:** https://arxiv.org/abs/2610.06563 (2026-10-05, preprint).
- **Loop:** frozen proposer/executor; evolve executable harness alongside **verified feasible/impossible task–environment pairs** to diagnose failures. Revises the harness, evaluates and generates fresh environment challenges; co-evolving an environment is still **H2** so long as neural weights are unchanged.
- **Held-out:** in a **60-pair** test set, abstention **61.7→83.3%**, feasible-task completion **68.3→76.7%** and paired success **48.3→70.0%**. A simpler 'generate from initial failures' baseline gets **73.3% paired success**, however, *better* than HERA on that conjunctive metric. Harness transfer to 19 other fixed models is reported; model and search budgets may differ.
- **Limits:** small generated environments and author-controlled verifiers, limited evidence for real-world feasibility judgments, hidden costs of task generation, and uncertain transfer beyond related simulator/task families. Record **all metrics**, not only the winning abstention figure.

### Scale and Selection for Visual-Interface Robot Agents — Zhijie Wei et al.

- **Primary:** https://arxiv.org/abs/2609.39304 (2026-09-30, v4 2026-10-22). An important **classification boundary**.
- **Clean H2 stage:** frozen executor and optimizer, repeated patch proposals to an executable screenshot-to-action harness, challenger/incumbent tests on identical cases and strict promotion across **30 rounds**. Batch size up to **100** produces **51→67%** success on **100 disjoint held-out LIBERO rollouts**. Unconditional acceptance in low-data settings can regress **42→34%** over 10 rounds; larger batches also spend more compute.
- **Separate trained-policy stage:** the work later distills a learned VLA executor from evolved-harness trajectories, reporting RoboCasa365 **62.2%** for the distilled policy and **71.2%** when combined with the harness. These results **must not be counted as frozen-weight H2**. Determine H4 status from the actual *coupling*, rather than assuming all stages are joint training.
- **Limitations:** simulated robotics, small held-out samples, repeated training-case reuse, rollouts cost and robustness to a changed task distribution.

## Established mechanism anchors (earlier papers, newly organized for this category)

### C. Agentic Harness Engineering (AHE) — Jiahang Lin et al.
- **Source:** https://arxiv.org/abs/2604.25850 (2026-04-28, revised 2026-05-18, v4).
- **Mechanism:** autonomous edits to file-exposed harness components, compressed trajectory/experience observability and predictions paired with subsequent task outcomes; edits are revertible. **No learned evolver/actor weight update** in the reported harness optimization.
- **Evidence:** Terminal-Bench 2 pass@1 **69.7% → 77.0%** after ten iterations; paper reports transfer across SWE-bench Verified and three alternate model families (approximately +5.1 to +10.1 pp on Terminal-Bench 2). Ablations favor tools, middleware and memory over prompt-only revisions.
- **Limitation:** controlled coding workloads and specific candidate-evaluation budget; proper matched token/rollout accounting and hidden test selection are important.

### D. Meta-Harness: End-to-End Optimization of Model Harnesses — Yoonho Lee et al.
- **Source:** https://arxiv.org/abs/2603.28052 (2026-03-30, arXiv).
- **Mechanism:** model proposes edits to executable context-management/harness code while consulting a filesystem with **full candidate code, scores and raw traces**. Neural weights frozen.
- **Evidence:** +7.7 accuracy points over reference context-management system on online text classification while using about four times fewer context tokens; +4.7 points average accuracy on 200 IMO-level retrieval-augmented math problems across five held-out models; also tests coding.
- **Ablation:** access to full traces substantially beats scores-only or LLM-generated summary conditions; supports evidence availability as a design decision, not only the prompt.
- **Limitation:** different benchmarks/compute units; best-of-N search and proposer capability must be matched.

### E. Darwin Gödel Machine (DGM) — Jenny Zhang et al.; A Self-Improving Coding Agent (SICA) — Maxime Robeyns et al.
- **Primary sources:** https://arxiv.org/abs/2505.22954 (DGM, 2025-05-29); https://arxiv.org/abs/2504.15228 (SICA, 2025-04-21).
- **Mechanisms:** SICA directly edits agent implementation; DGM uses an **archive/tree** of self-modified coding agent implementations and open-ended branch sampling. **Model weights stay fixed**; the implementation evolves.
- **Reported DGM results:** SWE-bench **20.0% → 50.0%**, Polyglot **14.2% → 30.7%** in its stated setup. SICA reported **17% → 53%** on a random SWE-bench Verified subset.
- **Limitations:** benchmarks/subsets, model and executor budget vary; uncontrolled code mutation, potential reward hacking; do not compare these headline gains to 2026 papers unless task/model/compute are matched.

### F. Retrospective Harness Optimization (RHO) — Wenbo Pan et al., Microsoft Research et al.
- **Source:** https://arxiv.org/abs/2606.05922 (first 2026-06-04; revised Aug 2026).
- **Mechanism:** retrieve difficult historical tasks, rerun them, propose edits, and select via pairwise **self-preference** without external grading at optimization time. Persistent harness state updated; weights fixed.
- **Reported evidence:** **59% → 78%** SWE-bench Pro success after one retrospective optimization pass in reported evaluation.
- **Limitations:** self-preference can be biased, vulnerable to correlated model errors; unclear representativeness of long-run gains under shifting task distributions. One cycle is weaker evidence for indefinite self-improvement.

### G. Additional boundaries and benchmark
- [Hierarchical Self-Improvement (HSI)](https://arxiv.org/abs/2608.08466) (Tailin Zhou, 2026-08): frozen LLM modifies harness **and the evolver's strategy code**, not neural weights; thus **H2**, despite the term "meta-evolver".
- [Living-Harness](https://arxiv.org/abs/2607.26598) (Yuetian Du et al., 2026-07): persistent episodic memory/state-graph repairs with fixed model/tools; an H2 **restricted-state** form, not full harness source-code rewriting.
- [EvoHarnessBench](https://arxiv.org/abs/2609.04280) (Salesforce Research et al., 2026-09): **benchmark**, not an evolution algorithm; 17 evolution streams / 802 tasks / 520 tools / 42 skills / 62 agents in original paper; documents harness-induced forgetting and adaptation–retention tension.
- [Beyond Prompts / PRISM](https://arxiv.org/abs/2609.05736) is a related restricted search over prompt and tool-boundary middleware, covered primarily in [H1](../h1-harness-engineering/research.md).
- [Evo²](https://arxiv.org/abs/2610.13025) has an automated harness search **plus learned evolver weights**, hence its fully trained configuration is **H3**, not H2. Its frozen-evolver ablation is an H2-compatible comparison.

## Contradictions / unresolved research questions

| Claim | Evidence | Countercondition |
| --- | --- | --- |
| The harness can autonomously improve without retraining | AHE, DGM, Meta-Harness, RHO, UCoE | Retention regressions; outcome judge errors; task-distribution dependence |
| Longer evolution guarantees monotonic gain | Positive selected branches in benchmark settings | EvoHarnessBench forgetting; selection on noisy evaluation; code-version drift |
| Self-preference solves missing labels | RHO and UCoE provide possible selection signals | Correlated judge biases, validator drift, feedback contamination |
| Source-code change and external-memory change are the same | Both can improve tasks | Distinct action space, portability, verifiability and failure modes |
| Evolution beats spending the same budget on task attempts | DGM/AHE/SSR improve selected benchmark metrics | *Rethinking Evaluation*: equal average held-out performance and stronger parallel task sampling | Publish train/selection/test splits and matched total feedback/rollout/compute budgets. |
| Strong champion–challenger selection ensures retention | SSR and robot experiments reduce noisy candidate promotion | *Continual Learners* shows forgetting and pre-block interference | Revisit historical tasks with rotating, independent retention tests. |
| Functional success implies security | Harmony can improve secure-pass on selected suites | In-sample grader exploitation and task-specific vulnerabilities | Independently evaluate exploit resistance and functional correctness on held-out threats. |
| More co-evolution always beats simpler curricula | HERA raises abstention and feasible-task completion | A simpler curriculum exceeds HERA's paired-success metric | Require conjunctive success, cost and transfer comparisons. |

**Next empirical priorities:** controlled repeated-deployment streams with holdout rotation; compute-normalized optimizer comparisons; tracking performance on *previously passed* tasks; external-judge audits; tool permissions and rollback for auto-generated code; fixed-evolver vs learned-evolver comparisons that cleanly expose H2↔H3 boundaries.

## Review history

2026-10-27: integrated SSR (Oct 20) and continual-learning analysis (Oct 19), plus earlier but missing counterevidence from *Rethinking Evaluation*, multiobjective Harmony, HERA, and the revised robot-harness study. Reweighted conclusions around **matched budgets, holdout transfer, retention, and secure admission**. Preserved the earlier H2 foundational literature; no changes to transversal, taxonomy or sibling files.

2026-10-14: first H2-focused systematic review, including the **2026-10-13 UCoE preprint** and **2026-10-12 Harness Self-Evolution working draft**. No prior specialized entries existed; comparisons from the human-owned parent research were used only as read-only context, not moved or deleted.
