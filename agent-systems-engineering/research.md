# Agent Systems Engineering — Living Research State

_Last reviewed: 2026-09-12_

This document is the curated research memory for end-to-end Agent Systems Engineering. It is organized by current technical conclusions rather than by weekly run chronology. Evidence is kept when it remains useful, and claims are revised when stronger or contradictory evidence appears.

## Current synthesis

### 1. Harness engineering is a real optimization surface, but the target is not a single static optimum

**Status: strengthened, with an important qualification.**

Multiple controlled studies now show that changing prompts, tool interfaces, middleware, state handling, and recovery logic while keeping model weights fixed can materially change agent performance. However, the newer evidence also shows that harness quality is distribution-, model-, workload-, and version-dependent. A harness that improves mean performance can be brittle, and adding capabilities to a harness can degrade tasks that were previously solved.

- **Airbnb — Beyond Prompts: Measuring and Optimizing LLM Tool-Agent Harnesses** (v2, 2026-09-09)  
  Primary source: https://arxiv.org/abs/2609.05736  
  PRISM optimizes prompts plus tool-boundary middleware around a fixed model. It reports mean held-out lifts of **+14.2 pp on BFCL multi-round, +14.9 pp on τ²-Retail, and +10.1 pp on τ²-Telecom**. More importantly, the paper shows that some search methods occasionally find large gains but still select brittle harnesses, motivating worst-condition lift, repeatability, and budgeted reliable-lift metrics.  
  **Implication:** harness search itself must be evaluated as a stochastic deployment process, not only by the score of its best candidate.  
  **Limitations:** the editable middleware surface is intentionally restricted to locally checkable tool-boundary patterns; it does not cover arbitrary control-flow rewrites, asynchronous state, or general graph redesign. Results use specific inner/outer model roles and benchmark simulators.

- **EVOHARNESSBENCH** (Salesforce Research et al., 2026-09-03)  
  Primary source: https://arxiv.org/abs/2609.04280  
  The benchmark contains **17 harness-evolution streams, 802 tasks, 520 tools, 42 skills, and 62 agents**. Merely expanding the harness can cause **harness-induced forgetting** on previously solved tasks. Retention and adaptation can also conflict: preserving old competence does not necessarily improve adaptation to newly introduced capabilities.  
  **Implication:** production harnesses should be treated as non-stationary systems requiring regression testing, compatibility/versioning discipline, and staged rollout—not as monotonic collections of tools and skills.  
  **Evidence quality:** strong benchmark design for this question, but still synthetic/benchmark-driven rather than a production postmortem.

- **Cursor — Continually improving our agent harness** (2026-04-30)  
  Primary source: https://cursor.com/blog/continually-improving-agent-harness  
  Cursor describes per-model harness customization, online/offline instrumentation, anomaly detection, and production tool-call reliability work. It reports driving tool calls to roughly **2–3 nines of reliability** and reducing unexpected tool-call errors by roughly **an order of magnitude** during a focused sprint.  
  **Implication:** industrial practice already looks closer to continuous systems optimization than one-time prompt engineering.  
  **Evidence quality:** production engineering report with concrete operational metrics, but not independently reproducible.

**Current conclusion:** the useful abstraction is not “find the best harness,” but **maintain a model/workload/version-specific harness under continuous evaluation and regression control**.

---

### 2. Verification layers are only useful when their evidence is sufficiently independent of the claim being verified

**Status: newly strengthened.**

A recurring failure mode is that a verifier, critic, or auditor receives upstream conclusions together with evidence and ends up relaying the conclusion rather than independently checking it.

- **Audit Without Verification: When LLM Accountability Layers Relay Rather Than Check** (2026-09-07)  
  Primary source: https://arxiv.org/abs/2609.07680  
  Pre-registered six-agent pipelines used process-level information boundaries, balanced defect injection, matched clean twins, **345,600 requests per chain model**, and two chain models. The pre-registered hypothesis that collective-responsibility framing degrades escalation with chain length was **not supported**, which is itself useful negative evidence.  
  The accountability layer nevertheless performed poorly when upstream reports contained their own conclusions. In clean episodes containing false alarms, it named an innocent party in **34.4% and 62.6%** of cases for the two chain models. When no agent proposed the true origin, an auditor recovered it from filed reports in only **4.1%** of cases, compared with **60.3%** when reading the raw documentation. Removing the agents’ conclusion field raised accuracy to **45.2% (+41.2 pp)** in that condition, while hurting cases where the upstream suggestion was actually correct.  
  **Implication:** an “LLM judge,” critic, compliance agent, or review agent should not be treated as independent simply because it is a different call or role. Independence has to exist in the **evidence path**.  
  **Limitations:** controlled pipeline rather than a deployed production system; conclusions depend on the specific reporting structure.

- **Anthropic — How we built our multi-agent research system** (2025-06-13, retained baseline)  
  Primary source: https://www.anthropic.com/engineering/multi-agent-research-system  
  Anthropic reported that an LLM judge worked well for its research-output evals when supplied with a strong rubric, but also retained human evaluation because automated grading misses source-quality and unusual failure modes.  
  **Reconciliation:** LLM judging is not generally invalid. The new result narrows the condition: it is dangerous when the judge is asked to verify a conclusion using evidence that already embeds or anchors on that conclusion.

**Current conclusion:** verification should maximize **evidence-path independence**, not merely role separation.

---

### 3. Authorization and safety-critical policy should be enforced at the mutation boundary, not left only in planner-visible context

**Status: newly strengthened.**

- **Beyond Agent Harnesses: Cross-Substrate Authority for Multi-Agent Systems** (2026-09-08)  
  Primary source: https://arxiv.org/abs/2609.08472  
  The paper studies a “cross-substrate authority gap”: authorization-relevant state can live outside the workspace/memory visible to a planner. In a 128-cell evidence ablation, authority-blind candidate evidence achieved **0/32** final semantic success, while raw authorization receipts and a typed relation each achieved **32/32**. Yet adding the typed relation to planning remained unreliable in a separate planning experiment. Replaying fixed model-generated first-action intents through a deterministic execution guard prevented **all six unsafe intents** and allowed **all 12 valid authorized publish intents**.  
  **Implication:** memory/context augmentation is not a substitute for deterministic authorization at the side-effect boundary. A correct architecture separates “what the model believes it may do” from “what the runtime will actually permit.”  
  **Limitations:** small controlled mini-benchmarks and narrow authority tasks; the result is architectural evidence, not proof of universal effectiveness.

- **Anthropic — How we contain Claude across products** (2026-05-25, retained industrial baseline)  
  Primary source: https://www.anthropic.com/engineering/how-we-contain-claude  
  Anthropic reports that users approved roughly **93%** of Claude Code permission prompts, motivating stronger environment-layer containment. Its native sandbox reduced permission prompts by **84%**. The company explicitly argues that probabilistic/model-layer defenses should not stand alone and that hard filesystem/network/egress boundaries cap blast radius. It also documents failures in custom allowlist/proxy logic despite underlying sandbox primitives working as designed.  
  **Implication:** the academic result above converges with production security practice: authority should be encoded in deterministic infrastructure wherever possible.

**Current conclusion:** for consequential side effects, **policy enforcement belongs below the reasoning layer**.

---

### 4. More agents and more generative orchestration do not automatically improve compound systems

**Status: refined; strong boundary conditions now visible.**

- **UnitBoost: Managing Compound LLM Systems with a Merge Operator, Not a Model** (2026-09-09)  
  Primary source: https://arxiv.org/abs/2609.09815  
  UnitBoost replaces a generative meta-manager with a deterministic merge operator over task-defined units. Across three held-out benchmarks it beats input-matched generative managers by **0.048–0.076 absolute task-score points**; replacing only the management step improves six compound-system configurations by **0.013–0.182**. Residual-directed rounds raise FanOutQA cell F1 from **0.4778 to 0.5524**.  
  The authors also identify explicit no-gain conditions: indivisible units, unavailable unit identity, and pricing that charges for every emitted unit.  
  **Implication:** some orchestration decisions become easier to reason about, test, and make order-invariant when moved out of an LLM and into deterministic operators.  
  **Limitations:** the method depends on decomposable task structure and a meaningful unit map; it does not eliminate the need for generative coordination on open-ended tasks.

- **At Equal Inference Cost, Multi-Agent Structure Does Not Beat a Single Frozen Agent** (submitted 2026-06-25; retained counterevidence)  
  Primary source: https://arxiv.org/abs/2609.04217  
  Under a fixed language-model call budget, a Planner–Executor–Critic team is not statistically better than an evolved single executor on ALFWorld (**0.769 vs. 0.754, p=0.80**) despite higher evaluation cost; the realized value comes almost entirely from the executor. WebShop shows no useful evolution and the team trends worse.  
  **Implication:** multi-agent gains must be compared at equal inference cost, not only equal environment rollouts.

- **Anthropic — How we built our multi-agent research system** (2025-06-13, retained positive counterpoint)  
  Primary source: https://www.anthropic.com/engineering/multi-agent-research-system  
  Anthropic reported **90.2%** improvement over single-agent Claude Opus 4 on an internal research eval using a lead Opus plus Sonnet subagents, while noting that multi-agent systems used about **15×** the tokens of chat and worked best on breadth-first, highly parallelizable research. Anthropic explicitly warns that coding and tightly coupled tasks are poorer fits.  
  **Reconciliation:** these results are not actually inconsistent. Multi-agent systems appear most justified when the task is **decomposable, parallelizable, and high-value enough to pay the compute/coordination overhead**. Under equal inference cost on tightly coupled environments, extra agent roles can add little or nothing.

**Current conclusion:** model count is not the key variable. The useful predictors are **task decomposability, shared-state coupling, inference budget, and whether orchestration can be made deterministic**.

---

### 5. AgentOps has a workflow-level scheduling problem above ordinary model serving

**Status: new and high priority.**

- **Decoupling Readiness from Release for Tail-Aware Scheduling of Agentic LLM Workflows** (2026-09-10)  
  Primary source: https://arxiv.org/abs/2609.10964  
  Most runtimes release each LLM turn to the inference engine as soon as dependencies are satisfied. The paper argues that under contention this eagerly commits work that can no longer be reordered at workflow level, increasing tail latency. It separates “ready” from “released” turns and schedules using a mean-CVaR objective plus an adaptive committed-work budget.  
  Evaluation uses closed-loop replay of real mini-swe-agent trajectories from **100 SWE-bench tasks (1,597 LLM turns)** and **100 SWE-Gym tasks (1,624 turns)** across Qwen3-8B, Qwen3-32B, and Llama-3.3-70B with vLLM. Under congestion, the method reduces workflow P95 flow time by up to **71.4% (3.5× speedup)** while remaining comparable to eager release under light load.  
  **Implication:** once agents become long-running workflows, serving optimization is not enough. The runtime needs a scheduler that reasons over **workflow state, dependency structure, committed work, and tail risk**. This is a concrete AgentOps layer distinct from model inference scheduling.  
  **Limitations:** matched closed-loop replay rather than a live multi-tenant production deployment; fixed traces mean behavior does not adapt to changed latency outcomes.

- **Avatar: Toward Autonomous End-to-End Orchestration of Scientific Workflows using LLMs** (2026-09-09)  
  Primary source: https://arxiv.org/abs/2609.10509  
  Avatar uses actor-separated orchestrator, executor, and provenance monitor roles with rule-based or LLM-backed policies behind an adapter-validated action catalog. Across three workloads, the LLM-backed version reports **55% less compute wastage** and **40% less GPU-busy time**.  
  **Implication:** adaptive agentic orchestration may improve resource utilization, but the evidence is currently much thinner than the tail-scheduling paper.  
  **Status:** promising but **deprioritized pending larger/independent evaluation**.

**Current conclusion:** AgentOps should include not only tracing and incident response, but also **workflow-aware admission, release, prioritization, and resource scheduling**.

---

## Contradiction / boundary-condition matrix

| Question | Evidence for | Counterevidence / boundary | Current interpretation |
|---|---|---|---|
| Does harness optimization matter with model weights fixed? | Airbnb PRISM: +10.1 to +14.9 pp held-out lift. Cursor reports major tool-reliability gains from harness work. | EVOHARNESSBENCH: expanding the harness can itself cause forgetting; PRISM shows optimizer-selected updates can be brittle. | **Yes, but deployment requires repeatability, regression control, and version-aware evaluation.** |
| Is a separate LLM verifier/auditor enough? | Anthropic successfully uses LLM judges for structured research-output evals. | Audit Without Verification: auditors can relay upstream conclusions; raw evidence can outperform report-mediated evidence by a large margin. | **Role separation is insufficient; evidence-path independence matters.** |
| Should permission/authority facts be put into the model context? | More planner-visible state can improve decisions. | Cross-Substrate Authority: planning remains unreliable; deterministic execution guard blocks all tested unsafe fixed intents. Anthropic production guidance prioritizes environment containment. | **Expose useful context, but enforce consequential authority at the mutation boundary.** |
| Does multi-agent structure improve capability? | Anthropic research: +90.2% internal eval, up to 90% speed reduction on highly parallel research. | MA-Evolve: no significant gain at equal inference cost on ALFWorld; WebShop trends worse. UnitBoost beats generative managers with deterministic merging on decomposable tasks. | **Benefit depends on decomposability, coupling, token budget, and manager necessity—not agent count.** |
| Should every ready agent turn be sent immediately to inference? | Eager release is simple and near-optimal under light load. | Tail-aware scheduling: up to 71.4% lower P95 under contention by retaining workflow-level scheduling optionality. | **Readiness and release should be decoupled in congested multi-workflow runtimes.** |

---

## Production engineering principles currently supported by the evidence

1. **Treat the harness as versioned production code, not prompt text.** Every change can alter capability, cost, reliability, and previously working behavior.
2. **Separate observation from enforcement.** The model may reason about permissions; the runtime should enforce them deterministically at the side-effect boundary.
3. **Design independent verification paths.** A critic that receives another agent's conclusion is not an independent verifier merely because it has a different role name.
4. **Price coordination explicitly.** Multi-agent systems should be compared under equal or at least reported inference/token budgets.
5. **Prefer deterministic orchestration where task structure allows it.** Merge, admission, routing, authorization, and stopping logic need not always be generative.
6. **Measure workflow-level SLOs.** Tail completion time, side-effect correctness, retries, recovery, cost per successful workflow, and state consistency are often more operationally meaningful than individual model-call latency.
7. **Expect non-stationarity at the harness layer.** New tools/skills/agents can break old behavior; regression suites and staged deployment are mandatory.

---

## Watch list / unresolved questions

- **Harness portability:** how much of a harness transfers across model families versus requiring co-design with the model?
- **Live production scheduling:** do workflow-aware release policies retain their P95 benefits under adaptive agents, heterogeneous tool latency, and real multi-tenant inference clusters?
- **Verifier independence:** what concrete architectural patterns best prevent critic/judge anchoring while controlling cost?
- **Durable side effects:** stronger evidence is still needed on idempotency, exactly-once/at-least-once semantics, compensation, and rollback for agents mutating external systems.
- **Canarying agent behavior:** conventional software canaries compare deterministic versions; agent canaries need distributional behavior metrics and state-aware rollback criteria.
- **Multi-agent shared-state design:** the boundary between useful parallelism and coordination collapse remains workload-specific; more production-scale head-to-head studies are needed.

---

## Compact retained history

These sources remain useful context but are not the main new evidence in the latest review.

- Anthropic, **How we contain Claude across products** (2026-05-25): https://www.anthropic.com/engineering/how-we-contain-claude — strong industrial evidence for environment-layer containment, sandboxing, egress controls, and limits of repeated human permission prompts.
- Anthropic, **How we built our multi-agent research system** (2025-06-13): https://www.anthropic.com/engineering/multi-agent-research-system — strong positive multi-agent production case, explicitly limited to parallelizable/high-value tasks and high token budgets.
- Cursor, **Continually improving our agent harness** (2026-04-30): https://cursor.com/blog/continually-improving-agent-harness — production harness observability, A/B testing, tool-error taxonomy, and per-model customization.
- OpenAI, **Introducing the Agents API** (2026-09-10): https://openai.com/index/introducing-the-agents-api/ — important evidence of industry convergence toward managed harness + sandbox + durable-session infrastructure, but primarily a product release; efficacy claims should be treated below controlled benchmark evidence unless independently reproduced.

