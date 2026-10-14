# H1 — Harness Engineering — Living Research State

_Last reviewed: 2026-10-14 (first focused review)_

**Classification:** H1 denotes harnesses designed or externally optimized around fixed executor weights, including operational engineering, code-level audits and controlled configuration comparisons. Automatic, open-ended harness evolution with fixed weights belongs primarily to [H2](../h2-self-evolving/research.md); evolver-weight learning belongs to [H3](../h3-weight-harness/research.md). This is a **scheduler-owned** document. The parent [research.md](../research.md) is reserved for human cross-category synthesis and must never be changed by this watch. Consult the [taxonomy](../taxonomy.md).

## Current synthesis

1. **Harness design is a distinct optimization and measurement axis (supported, but workload-specific).** With a fixed executor, changes to tool interfaces, prompts and tool-boundary middleware can materially affect held-out task success. Do not attribute system gains to a better language model when the weights did not change; equally, do not assume an optimized harness transfers across models.
2. **Optimization must select a reliably good harness, not merely discover one high-scoring candidate (supported in controlled settings).** Selection variance, worst-condition lift, regression gates and search budget matter more than peak development score.
3. **The unit of evaluation is the (model, harness, workload, budget) configuration (supported by diagnostic evidence).** Different harness/model pairs exhibit distinct execution-alignment failures, latency, tool costs, and state divergence.
4. **Production harnesses are operated as versioned software with observability (industrial evidence).** Model-specific tool protocols, tool-error classification, A/B tests, offline evals, canaries and rollback are engineering practices. Industry claims lack independent reproducibility and should not be conflated with controlled scientific evidence.
5. **Implementation patterns are converging, but general prescriptions remain tentative.** Large code audits provide a taxonomy of implementations, not proof that a pattern is optimal or that absence of a dependency causes superior results.

## Primary evidence (latest evidence available by 2026-10-14)

### A. Beyond Prompts: Measuring and Optimizing LLM Tool-Agent Harnesses — Airbnb (Cen Mia Zhao et al.)
- **Source:** https://arxiv.org/abs/2609.05736 (2026-09-04, v2 2026-09-09; research paper/preprint, arXiv metadata reports EMNLP 2026 acceptance).
- **Mechanism and weights:** constrained prompt/tool-boundary middleware edits with **fixed inner/executor model**. PRISM routes clusters of errors to prompt vs middleware changes; editable middleware comprises bounded intercept patterns, **not arbitrary rewriting of the agent loop**.
- **Evidence:** held-out average gains **+14.2 pp BFCL multi-round, +14.9 pp tau2-Retail, +10.1 pp tau2-Telecom**; introduces repeatability, worst-condition lift and selected-harness `RelLift95(B)` under fixed rollout budget. Component tests mainly credit repair-surface routing and edit-pattern constraints.
- **Interpretation:** defensible H1 measurement protocol, with an H2-adjacent automated optimizer inside a **restricted surface**. For taxonomy purposes treat the evaluation/engineering result as H1 and cross-link to H2, rather than asserting comprehensive self-rewriting runtime.
- **Limitations:** offline simulated tasks, small held-out scorecards and stochastic users; no multi-cycle production re-optimization. RelLift95 is an empirical diagnostic, not a formal future-deployment guarantee. Trial scoring and compute accounting need direct inspection.

### B. Harness-Bench: Measuring Harness Effects across Models in Realistic Agent Workflows — Yilun Yao et al.
- **Source:** https://arxiv.org/abs/2605.27922 (2026-05-27, arXiv preprint); code: https://github.com/Qihoo360/harness-bench .
- **Mechanism and weights:** vary harness *configuration* across model backends under common task environments, task oracles, resource envelopes and evaluation. No model or harness training in the benchmark.
- **Evidence:** the **paper's original version** reports **106 sandboxed tasks and 5,194 execution trajectories**, with systematic differences in completion, process quality and efficiency among pairings. A later project site describes a larger suite (300 tasks/20,454 runs); **do not mix this evolving website count with the versioned paper without matching the dataset version**.
- **Limitation:** representative configurations and offline fixtures, not isolation of the causal effect of any individual code component or proof of real deployment superiority. Important foundation for matched model–harness comparisons.

### C. Harness Engineering: Anatomy, Architecture, and Evolution of Coding Agents — Paul Barbaste et al.
- **Source:** https://arxiv.org/abs/2609.00006 (2026 paper / versioned source-code study).
- **Mechanism:** reverse engineering of **11 coding-agent harnesses**, documenting canonical subsystems, **29 recurring patterns**, and a longitudinal source-diff sample; no model training, no optimization experiment.
- **Evidence:** across the studied corpus, agents use custom async loops and deterministic retrieval rather than general-purpose agentic framework imports or vector embedding retrieval. This is a **corpus observation**, *not* a universal claim about all coding agents or a causal statement about framework quality.
- **Limitation:** release pinning, inclusion criteria, hidden/proprietary systems and versions constrain generalization. Useful as an architectural atlas and repository-reading guide.

### D. Continually improving our agent harness — Stefan Heule & Jediah Katz, Cursor
- **Source:** https://cursor.com/blog/continually-improving-agent-harness (2026-04-30; production engineering report).
- **Mechanism:** per-model harness customization, tool-error taxonomy, model-specific interfaces, offline benchmark + live A/B instrumentation, keep-rate and anomaly alerts, ongoing context-selection redesign.
- **Reported evidence:** tool reliability raised to roughly **2–3 nines**, and unexpected tool-call errors reduced by roughly **one order of magnitude** in a focused sprint. An expensive summarizer offered negligible quality improvement and was not deployed; the report explicitly cautions against equating extra model spend with benefit.
- **Limitation:** internal metrics and experiments, not independently replicable; operational tool reliability is not the same as end-to-end task success.

### E. Supporting system-boundary evidence (retained from pre-split research)
- [Audit Without Verification](https://arxiv.org/abs/2609.07680) (2026-09): an LLM auditor can propagate upstream claims unless its evidence path is sufficiently independent. H1 relevance: verification architecture; **not** evidence that additional model roles alone help.
- [Beyond Agent Harnesses: Cross-Substrate Authority](https://arxiv.org/abs/2609.08472) (2026-09) and [Anthropic: How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude) (2026-05): deterministic authorization and sandbox constraints at execution boundaries; narrower controlled tests + production account, respectively.
- [EvoHarnessBench](https://arxiv.org/abs/2609.04280) (2026-09): expanding tools/skills may cause **harness-induced forgetting**; read the controlled benchmark details in H2's specialized synthesis before citing as a self-evolution algorithm.

## Contradictions / boundary conditions

| Question | Positive evidence | Negative or limiting evidence | Interpretation |
| --- | --- | --- | --- |
| Can harness changes help with weights fixed? | PRISM gains; Cursor operational metrics | PRISM small offline sets; retention regressions in EvoHarnessBench | Yes, with regression and cost controls. |
| Does an intrinsically stronger model remove harness sensitivity? | Some stronger models need fewer guardrails (Cursor) | Harness-Bench finds model–harness pairing effects; native tool contracts differ | Co-design for the actual model; no universal harness ranking. |
| Do extra prompts/agents improve reliability? | Directed prompt/middleware repairs help in PRISM | Auditor anchoring; unnecessary expensive summarization (Cursor) | Prefer failure-localized, testable changes and independent evidence paths. |
| Is an observed pattern a best practice? | Source-code audit finds common patterns | Observational sampling cannot identify causal performance effects | Use as hypotheses to test, not prescriptions. |

## Implementation/evaluation questions to track

- Per-model compatibility, **state and tool-call correctness**, rollback and model-switch semantics.
- Held-out changes after *multiple* consecutive harness updates; regressions on previously solved tasks.
- Task completion vs tool-call accuracy vs user satisfaction; cost per successful task, tokens, latency, tail reliability.
- Benchmarks that allow component-level causal isolation while preserving native agent-loop behavior.

## Review status and retained history

First H1-focused synthesis assembled on 2026-10-14 from existing parent research and primary sources. No claim of a **new October H1 breakthrough** is made merely because this specialized file was initialized. Historical multi-agent/AgentOps work continues to reside in the **untouched** parent document until the human researcher integrates it.
