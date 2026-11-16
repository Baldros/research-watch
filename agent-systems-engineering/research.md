# Agent Systems Engineering — Living Research State

_Last reviewed: 2026-11-16_

This document is the manually curated cross-category synthesis for end-to-end Agent Systems Engineering (see [taxonomy.md](./taxonomy.md) and the H1–H4 research files linked in [README.md](./README.md)). Its primary object is the **software system around the model**: context, tools, memory, harness, orchestration, execution environment, permissions, verification, and operations. Reported changes are attributed to the **model**, **harness/context**, **orchestration**, **runtime**, or **production operations**, rather than to an undifferentiated 'agent'. It is organized by live technical conclusions, not a chronological weekly append-only log. Retained findings can be strengthened, weakened, or superseded as evidence changes.

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


#### Additional controlled counterevidence and production evidence (October–November 2026)

- **Agency — System Optimizations for Cost-Efficient Agent Harnesses** (Park et al.; preprint 2026-11-09; https://arxiv.org/abs/2611.12122). **Layer: harness interposition / cost**, with frozen task models. On 100 coding tasks (20 Terminal-Bench 2, 40 SWE-bench, 40 SWE-bench Pro), the authors found broadly similar accuracy across Claude Code, Codex CLI, and intentionally minimalist harnesses for the tested frontier models, despite cost variation up to **2.6×** on fixed model/task. Their interposition platform manipulates context/tool/environment boundaries and reports **up to 35% end-to-end cost reduction** on benchmark runs with little accuracy loss. Trace analysis reports up to **73% of total cost** from context reads, and only **27% of tool outputs** consumed on average. **Counterevidence:** this does *not* establish universal harness irrelevance: PRISM's multi-turn tool benchmarks and HarnessFlow's optimization experiments measure different workloads and harness interventions, and [When Agents Fail](https://arxiv.org/abs/2610.32483) finds strong pairing-specific differences. **Limitations:** a sampled coding-task panel, evolving frontier model versions, controlled harness configurations, and cost-dependent provider pricing; do not extrapolate the null effect to all tool, web, multi-agent, and long-horizon domains. The cost optimization is a harness effect, not model training.

- **HarnessFlow — MCTS-Guided Harness Optimization for LLM Agents** (Hong et al.; preprint 2026-11-04; https://arxiv.org/abs/2611.06254). **Layer: H2 automated full-harness search**, not executor weight training. Keeps task model and environment fixed; edits complete executable harnesses via coding agents using a component library and a tree of prior candidates. Across six benchmark families (TAU2-bench, AppWorld, BigCodeBench-Hard, SWE-bench Verified, WebShop, BrowseComp-Plus), with **15 refinements per benchmark**, reports mean **69.48% test score**, **+10.89 percentage points vs initial harness** and **+3.95 points vs Meta-Harness**. Mean execution cost for optimized harnesses is reported at **41.4% of Codex** on this set. **Interpretation:** targeted full-harness edits can improve capability without changing executor weights, unlike the small harness-effect result in Agency's coding panel. **Limitations:** optimize-and-select search incurs a separate nontrivial exploration/evaluation budget; different baselines and workloads preclude direct numerical comparisons to Agency or PRISM. Check reproducibility and repeated-seed generalization.

- **Cursor — Improved token efficiency for longer agent runs** (engineering report with A/B testing, 2026-09-23; https://cursor.com/blog/improved-token-efficiency). **Layer: context assembly / tool disclosure / cache, not model weights.** Cursor reports a **7% reduction in user token cost without observed quality loss** after a *bundle* of harness changes: **66% shorter system prompt**, **60% fewer static tool-description tokens** after moving tools to dynamic loading, and **20% fewer cold-cache misses** after changing cache boundaries. A prior MCP-disclosure change reduced **total tokens 46.9% only for sessions that invoked an MCP tool**; line-number compression reduced **cache-read tokens 1.6%**. These denominators differ and are not additive. The team also *removed* instructions strongly encouraging codebase-exploration subagents because of coordination overhead. **Counterpoint:** rather than supporting ever-larger prompts or delegation trees, this provides production evidence of a *model-conditioned simplification* trend, consistent with EVOHARNESSBENCH's regressions and Agency's cost focus. **Limitations:** bundled interventions and a changing mix of models/tasks; no independently published factorial ablation or full confidence intervals.

**Revised conclusion [strengthened, conditioned]:** harness optimization has at least **two distinct objectives**: (a) task quality under a fixed model/workload and (b) cost/latency/reliability at a chosen quality floor. A small accuracy delta on mature coding models does not negate large deltas in tool-agent benchmarks or the value of runtime efficiency. Measure the interaction \(\mathrm{Outcome}=f(M,H,T,E,B)\), holding model \(M\), task distribution \(T\), environment \(E\), and budget \(B\) explicit.

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


- **Cross-check on delegation (Cursor, 2026-09-23):** https://cursor.com/blog/improved-token-efficiency — Cursor removed prompts that *over-encouraged* exploratory subagents: isolated child contexts can reduce repeated parent context but introduce duplication and coordination costs. This is compatible with Anthropic's positive breadth-first research result and with equal-budget negative multi-agent experiments: the boundary variable is not agent count but separability, useful parallel work, and whether coordination adds independently verifiable information. **Layer:** orchestration/context; evidence is production configuration/A-B practice, not a head-to-head universal winner.

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


#### Durable tool execution and state (new, high-priority evidence)

- **RECEIPT — Measuring and Repairing the Return Assumption of LLM Agents in Web Service Operations** (Yang et al.; preprint 2026-11-01; https://arxiv.org/abs/2611.01930). **Layer: tool-process I/O contract and runtime observability.** The study collects **91,469 production tool calls** and **89,047 public agent trajectories**, tests the process-launch semantics of **eight harnesses**, and constructs **158 checked CLI invocations** plus **25 paired tasks** where only an interactive question gates the desired action. **Seven of eight** harnesses close input in the nonterminal configuration; **11.5%** of tested invocations then report exit status 0 without performing the requested action. Keeping an interactive terminal open without responding multiplies task token use **2.9×**. RECEIPT observes system-call wait state and reports it to an unmodified agent, reducing **false success claims by 73.5% across five models**; on the second tested harness, it finishes all 25 guarded tasks. **Implication:** exit=0 is not a semantic completion receipt; a harness must expose *what the process did and is waiting for*. **Limitations:** curated command classes and task pairs, not every production shell/workload; control of stdin/TTY and tool behavior is the causal variable. **Counterpoint:** simple tool-loop baselines that look competent on standard benchmarks can fail through execution-interface details that models cannot infer reliably.

- **Anthropic — Scaling Managed Agents with stable interfaces** (engineering architecture report; https://www.anthropic.com/research/managed-agents). **Layer: session persistence + stateless harness + independent sandboxes.** Separates a durable append-only session/event log from replaceable harness processes and execution sandboxes; a failed harness can resume from `wake(sessionId)`, and sandbox failures appear as tool errors rather than destroying the whole agent session. Decoupling sandbox provisioning from inference produced reported **~60% lower p50 time-to-first-token** and **>90% lower p95**. **Relevance:** reliability improvements arise from storage and failure-domain design, *not* from changing the model. **Limitations:** industrial report, not an independently reproduced benchmark. Compare with alternative substrate designs only at the corresponding architectural layer.

- **Google — Agent Substrate / GKE and CNCF** (initial announcement 2026-09-15; **major status update 2026-11-05**). Primary sources: https://cloud.google.com/blog/products/containers-kubernetes/agent-substrate-available-on-gke ; https://cloud.google.com/blog/products/containers-kubernetes/open-infrastructure-for-autonomous-agents-comes-to-cncf/ ; https://github.com/agent-substrate/substrate . **Layer: execution substrate, actor scheduling and sandbox security.** Maps many stateful agent actors onto fewer workers, snapshots RAM/filesystem state across inactivity, uses gVisor or microVM isolation, and places credential injection / default-deny network enforcement outside untrusted execution. Reported **<500 ms resumes**, **>500 activations/s** and roughly **10× container density** under workload-specific measurement. **Status revision [prior early-development caution superseded in part]:** in November Google announced **Agent Substrate 1.0 and GA on GKE**, and Agent Substrate, gVisor and OpenRL were accepted into **CNCF Sandbox**; the current repository claims compatibility for 1.0 public APIs. Earlier September upstream statements that the project was not production-ready are now stale. **Caveat:** GA, API stability, and CNCF Sandbox membership are maturity/governance markers, not independent proof that all claimed densities or isolation guarantees hold under arbitrary workloads. Agent Substrate is **not itself an agent SDK or planning harness**. Its resource-life-cycle optimizations complement, rather than replace, the workflow-level inference scheduler analyzed above.

- **Grab — How we help Grab build and run AI agents at scale** (2026-07-24; newly integrated production baseline; https://engineering.grab.com/how-grab-builds-and-runs-ai-agents-at-scale). **Layer: enterprise AgentOps paved road.** Reports **500+ production agent services**, **50+ remote MCP servers**, and billions of model tokens/month through a central gateway. Building the basic model/tool loop took an afternoon; configuring auth, OIDC, Vault, tracing, service deployment, health checks, and persistence took about **two weeks**. Its LLM-Kit template and shared eval/observability tools reduced initial wiring to **about one hour**. **Implication:** much of AgentOps is applying established infrastructure/SRE primitives consistently across probabilistic workflows, not inventing a new type of graph. **Limitations:** organization-specific case study, no randomized across-team causality claim.

- **Cursor — Agent SDK and Self-Hosted Machines** (2026-10-20; 2026-09-02). Sources: https://cursor.com/blog/agent-sdk ; https://cursor.com/docs/cloud-agent/self-hosted . **Layer: agent-as-code packaging/deployment, remote execution placement and trust boundary.** Agent SDK expresses instructions, typed tools, skills, MCP connections, channels and evals as versioned project artifacts, with `agent-sdk validate`, local dev and managed `agent-sdk deploy`. Self-Hosted Machines runs edits/terminals/MCP locally on customer workers **but Cursor cloud still runs planning, agent loop and inference**; tool outputs (including code snippets/diffs/screenshots) can return to Cursor. **Implication:** self-hosted *tool execution* must not be represented as fully local inference or zero cloud data flow. This is source-verified implementation design, **not** a controlled performance or confidentiality comparison.

- **xAI — Grok Build harness/runtime changelog** (2026-09–11; https://x.ai/changelog/build ; https://github.com/xai-org/grok-build). **Layer: persistent-session and tool lifecycle, permissions and subagent runtime.** Documented fixes include retained compaction history, durable background-task state on reconnect, correct cancellation propagation for MCP calls, serializing simultaneous edits of one file, handling interrupted sessions, persistent shell restore behavior, and warnings when permission rules fail to parse (Nov 4). **Relevance:** implementation-level failure taxonomy for long-running agents; bugs in state restoration, cancellation, and policy loading change system safety even with identical model weights. **Limitations:** release notes document implemented changes, **not** their measured task impact or absolute production reliability; do not treat the release itself as a performance benchmark.

**Revised conclusion [strengthened]:** AgentOps comprises at least four separable technical planes—**workflow-level inference scheduling, tool/process execution semantics, stateful sandbox lifecycle, and organization-wide deployment/observability controls**. Every plane needs its own correctness and cost metrics. The executable interface can reverse an apparent model ranking without any weight change.

**Current conclusion:** AgentOps should include not only tracing and incident response, but also **workflow-aware admission, release, prioritization, and resource scheduling**.

---

### 6. Attribute outcomes to model–harness fit, not model or harness alone

**Status: new, strong direct comparative evidence.**

- **When Agents Fail: Model or Harness—Who's to Blame?** (Zhang et al.; preprint 2026-10-23; https://arxiv.org/abs/2610.32483). **Layer: interaction evaluation protocol.** Crosses **four general-purpose harnesses** (OpenCode, Codex CLI, Pi CLI, DeepAgents) with **four core models**, adds two fixed native systems and stress cases, yielding **111 evaluation settings / 52,168 trajectories** across six benchmark families. Introduces **Valid Completion Rate (VCR)** (normal completion and valid evaluator handoff, separate from correctness), model–harness fit, cross-model fit, pairing-specific interaction residuals, and token load including failed runs. Rankings change with the model and task; task success and valid completion can prefer different pairings. **Implication:** aggregate pass@1 should not be interpreted as a pure model capability measure; evaluate a **crossed design** (same task/model across several harnesses, same harness across several models) plus complete lifecycle validity.
  **Limitations:** benchmark/adapter choices matter; not every native feature can be normalized across systems. It does not prove one harness universally best.

- **Concrete apparent disagreement:** Agency (https://arxiv.org/abs/2611.12122) reports little accuracy impact from swapping some harnesses in a 100-task modern coding panel, whereas HarnessFlow (https://arxiv.org/abs/2611.06254) finds large gains by optimizing executable harnesses on six different benchmark distributions. These do **not** cancel each other: 'choose among mature/off-the-shelf or minimalist coding harnesses' is a different intervention from 'search/edit the full harness'; model choice, task difficulty, context/tool requirements, and optimization budget differ. PRISM (https://arxiv.org/abs/2609.05736) further shows substantial gains in multi-round tool tasks. **Current conclusion:** harness importance is a *conditional empirical question* and must be reported with workload, model, initial harness, intervention surface, budget, and failure categories.

---

## Contradiction / boundary-condition matrix

| Question | Evidence for | Counterevidence / boundary | Current interpretation |
|---|---|---|---|
| Does harness optimization matter with model weights fixed? | Airbnb PRISM: +10.1 to +14.9 pp held-out lift. Cursor reports major tool-reliability gains from harness work. | EVOHARNESSBENCH: expanding the harness can itself cause forgetting; PRISM shows optimizer-selected updates can be brittle. | **Yes, but deployment requires repeatability, regression control, and version-aware evaluation.** |
| Is a separate LLM verifier/auditor enough? | Anthropic successfully uses LLM judges for structured research-output evals. | Audit Without Verification: auditors can relay upstream conclusions; raw evidence can outperform report-mediated evidence by a large margin. | **Role separation is insufficient; evidence-path independence matters.** |
| Should permission/authority facts be put into the model context? | More planner-visible state can improve decisions. | Cross-Substrate Authority: planning remains unreliable; deterministic execution guard blocks all tested unsafe fixed intents. Anthropic production guidance prioritizes environment containment. | **Expose useful context, but enforce consequential authority at the mutation boundary.** |
| Does multi-agent structure improve capability? | Anthropic research: +90.2% internal eval, up to 90% speed reduction on highly parallel research. | MA-Evolve: no significant gain at equal inference cost on ALFWorld; WebShop trends worse. UnitBoost beats generative managers with deterministic merging on decomposable tasks. | **Benefit depends on decomposability, coupling, token budget, and manager necessity—not agent count.** |
| Should every ready agent turn be sent immediately to inference? | Eager release is simple and near-optimal under light load. | Tail-aware scheduling: up to 71.4% lower P95 under contention by retaining workflow-level scheduling optionality. | **Readiness and release should be decoupled in congested multi-workflow runtimes.** |

| Is harness choice irrelevant to task quality? | Agency finds similar coding-task accuracy across certain harnesses with new models; cost changes up to 2.6×. | PRISM +10–15 pp on multi-turn tool tasks; HarnessFlow +10.89 pp after full-harness search; model–harness fit varies over 52,168 trajectories. | **No universal answer. Control model, workload, harness intervention surface and search budget.** |
| Does a shell command's exit code certify that the action happened? | Ordinary shell loops infer completion from a returned status. | RECEIPT: 11.5% of test invocations exited 0 before the requested action; a runtime repair reduced false success reports 73.5%. | **Track semantic completion and interactive wait state, not just process return code.** |
| Does increasing harness context and delegation automatically help? | Rich prompts/tool catalogs and subagents can support less capable models and long workflows. | Cursor reduced prompt length 66%, static tool-description tokens 60%, and deliberately discouraged unnecessary exploratory subagents; EVOHARNESSBENCH documents regressions. | **Harness scaffolding is model- and workload-conditioned; minimize it only with quality-controlled measurement.** |
| Is the execution substrate just a container per agent? | Traditional Kubernetes handles worker fleet management and service restart. | Agent Substrate 1.0 snapshots dormant actors, multiplexes them over pre-warmed workers and enforces sandbox/egress policy; Anthropic decouples durable sessions and stateless harnesses. | **A separate agent-specific stateful runtime plane can be justified, though comparative operational data remain limited.** |
| Does self-hosted tool execution imply fully on-prem inference? | Cursor workers keep workspaces, commands and credentials on customer machines. | Cursor documents cloud-resident planning/inference and return of tool outputs, diffs and screenshots. | **No. Map the actual data flow and split the trust boundaries explicitly.** |

---

## Production engineering principles currently supported by the evidence

1. **Treat the harness as versioned production code, not prompt text.** Every change can alter capability, cost, reliability, and previously working behavior.
2. **Separate observation from enforcement.** The model may reason about permissions; the runtime should enforce them deterministically at the side-effect boundary.
3. **Design independent verification paths.** A critic that receives another agent's conclusion is not an independent verifier merely because it has a different role name.
4. **Price coordination explicitly.** Multi-agent systems should be compared under equal or at least reported inference/token budgets.
5. **Prefer deterministic orchestration where task structure allows it.** Merge, admission, routing, authorization, and stopping logic need not always be generative.
6. **Measure workflow-level SLOs.** Tail completion time, side-effect correctness, retries, recovery, cost per successful workflow, and state consistency are often more operationally meaningful than individual model-call latency.
7. **Expect non-stationarity at the harness layer.** New tools/skills/agents can break old behavior; regression suites and staged deployment are mandatory.
8. **Treat execution as a protocol, not a string-returning black box.** Check for unfinished interactive commands, environment side effects, timeouts and cancellation acknowledgement, even if a process exits successfully.
9. **Use crossed model × harness evaluations.** Report success and valid completion separately and include cost and failed attempt traces; compare quality gains and cost gains as distinct objectives.
10. **Version agents like production software.** Packaging, policy, tool schemas, snapshots, eval fixtures, canarying and rollback should have explicit compatibility and reproducibility contracts.
11. **Keep cloud-vs-local boundaries precise.** Distinguish placement of inference, orchestration, tools, credentials, workspace and telemetry rather than using 'self-hosted' as a security claim.

---

## Watch list / unresolved questions

- **Harness portability:** how much of a harness transfers across model families versus requiring co-design with the model?
- **Live production scheduling:** do workflow-aware release policies retain their P95 benefits under adaptive agents, heterogeneous tool latency, and real multi-tenant inference clusters?
- **Verifier independence:** what concrete architectural patterns best prevent critic/judge anchoring while controlling cost?
- **Durable side effects:** stronger evidence is still needed on idempotency, exactly-once/at-least-once semantics, compensation, and rollback for agents mutating external systems.
- **Canarying agent behavior:** conventional software canaries compare deterministic versions; agent canaries need distributional behavior metrics and state-aware rollback criteria.
- **Multi-agent shared-state design:** the boundary between useful parallelism and coordination collapse remains workload-specific; more production-scale head-to-head studies are needed.
- **Agency vs PRISM vs HarnessFlow:** reproduce the conflicting harness-sensitivity findings with fixed model versions and common initial harnesses, equal task sets, held-out seeds and total *optimization* budgets.
- **RECEIPT runtime portability:** test process stdin/TTY waiting, acknowledged side effects and cancellation correctness across coding, web and workflow agents, including MCP remote tools.
- **Agent Substrate 1.0 / CNCF:** measure restored-state correctness, density, egress isolation and p95 resume under realistic noisy-neighbor/failure conditions independently of vendor benchmarks.
- **xAI Grok Build:** seek audited harness-vs-model evidence, not just changelog feature counts, for persistent agent reliability and subagent use.
- **Cursor deployment data flow:** assess cloud-planner/local-executor boundaries, tool-output redaction, persistent credentials, workspace artifacts, and actual data movement in external audits.

---

## Compact retained history

These sources remain useful context but are not the main new evidence in the latest review.

- Anthropic, **How we contain Claude across products** (2026-05-25): https://www.anthropic.com/engineering/how-we-contain-claude — strong industrial evidence for environment-layer containment, sandboxing, egress controls, and limits of repeated human permission prompts.
- Anthropic, **How we built our multi-agent research system** (2025-06-13): https://www.anthropic.com/engineering/multi-agent-research-system — strong positive multi-agent production case, explicitly limited to parallelizable/high-value tasks and high token budgets.
- Cursor, **Continually improving our agent harness** (2026-04-30): https://cursor.com/blog/continually-improving-agent-harness — production harness observability, A/B testing, tool-error taxonomy, and per-model customization.
- OpenAI, **Introducing the Agents API** (2026-09-10): https://openai.com/index/introducing-the-agents-api/ — important evidence of industry convergence toward managed harness + sandbox + durable-session infrastructure, but primarily a product release; efficacy claims should be treated below controlled benchmark evidence unless independently reproduced.

