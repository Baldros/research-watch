# H2 — Self-Evolving Harness — Living Research State

_Last reviewed: 2026-10-14 (first focused review)_

**Category:** a system iteratively rewrites externally stored harness artifacts based on execution feedback while **executor and optimizer/evolver neural weights remain fixed**. A meta-evolver that modifies the *evolver's source code* but does not train its weights remains H2. Exclude evolver-weight learning ([H3](../h3-weight-harness/research.md)) and executor-weight co-adaptation ([H4](../h4-co-adaptation/research.md)). See [taxonomy](../taxonomy.md). This is the scheduler's **only** writable research file; [parent research.md](../research.md) is human-owned/read-only.

## Current synthesis

1. **Closed-loop harness search is a practical alternative to changing neural weights** (supported by multiple experimental preprints). The editable artifact ranges from task instructions/procedural memory to runtime code and orchestration. These ranges should *not* be treated as equivalent.
2. **Search-history access, error attribution, and guarded candidate selection appear as recurring mechanisms** (multiple independent papers; cross-study numbers are not directly comparable). An editable program may improve accuracy, but also introduces code-execution and regression risks.
3. **A harness may improve on new tasks while forgetting previous ones.** Harness evolution is a non-monotonic systems change. Longitudinal retention, held-out transfer and clean separation of discovery from final evaluation remain essential.
4. **Self-judgment is an imperfect substitute for external truth.** Retrospective algorithms and evolving verifier programs offer ways to work without labels; they can also reinforce verifier errors and contaminate persistent procedural memory.
5. **2026-10-12 survey (working draft) identifies a substantial evidence gap.** Many claims have only single-round task gains, with weak activation/attribution demonstrations. The audit is useful as a *methodology proposal*, not conclusive independent validation.

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

**Next empirical priorities:** controlled repeated-deployment streams with holdout rotation; compute-normalized optimizer comparisons; tracking performance on *previously passed* tasks; external-judge audits; tool permissions and rollback for auto-generated code; fixed-evolver vs learned-evolver comparisons that cleanly expose H2↔H3 boundaries.

## Review history

2026-10-14: first H2-focused systematic review, including the **2026-10-13 UCoE preprint** and **2026-10-12 Harness Self-Evolution working draft**. No prior specialized entries existed; comparisons from the human-owned parent research were used only as read-only context, not moved or deleted.
