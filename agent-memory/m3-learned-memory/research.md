# M3 — Learned Memory & Model–Memory Co-Adaptation — Living Research State

_Reviewed: 2026-10-20. First extended evidence pass. This scheduler owns **only this file**; parent [research.md](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md) and sibling files remain read-only._

## Current synthesis

1. **“Learned memory” does not imply updating the backbone that answers requests.** Memorilla trains a reader projecting external document embeddings into a frozen executor's input space; JAM trains a separate Researcher; MemoPilot trains a memory-update copilot for a frozen player. CLM, Memory-R1 and UMA also investigate learned context/memory operations, with different trainable components. These are not interchangeable architectures. [N1–N6]
2. **There are at least three different learned decisions:** *(a)* compress/represent stored evidence into tokens (Memorilla), *(b)* navigate a source-preserving external store and choose query-time evidence (JAM, NapMem), *(c)* decide how to create/update/delete memory over a stream (Memory-R1, UMA, MemoPilot, Memory-R2). Judge their contribution against the matching non-learning baseline. [N1, N3–N7]
3. **Training a memory agent is a long-horizon RL problem with nonstationary state caused by its own actions.** Memory-R2 observes that different rollout branches no longer share an equivalent memory history; its local rerollouts offer a principled response to unfair group-relative credit assignment. It is still an author-reported preprint result. [N8]
4. **Model-controlled files are not necessarily trained policies.** CLM includes zero-shot context-file use, instruction search and parametric RL variants. Only the learned variants are strong M3 exemplars; plain file management belongs mainly to M2/H1. [N2]
5. **The strongest remaining uncertainty is cost-matched generalization.** Token reductions can hide document encoding, repeated tool calls, attention passes, index storage, train-time rollouts and extra model invocations. Several benchmark comparisons use different backbone sizes/checkpoints; those are evidence of feasibility, not clean causal estimates.

## Mechanism / trainability map

| System | Persistent substrate | Who controls reading/writing? | What is trained in the studied configuration? |
| --- | --- | --- | --- |
| **Memorilla [N1]** | Encoded document vectors, converted to a few memory tokens at query time | Query-conditioned neural memory module | Trainable memory module; **LLM backbone frozen** |
| **CLM [N2]** | Mutable context file, possibly across agents | Executor acting on context; prompt or learned policy | **Variant-dependent**: zero-shot none, instruction optimization external, RL trains policy |
| **JAM [N3]** | Raw hierarchical page-store with summaries | Agentic Researcher opens, searches and browses | Researcher via SFT then Hint-guided GRPO; Memorizer held fixed during this training |
| **Memory-R1 [N4]** | Structured external memory bank | Manager ADD/UPDATE/DELETE/NOOP; answer policy selects records | Both roles fine-tuned with PPO/GRPO in the paper |
| **UMA [N5]** | Compact core summary + key/value Memory Bank | Unified stream-memory and answering policy | Policy trained end to end with RL; explicit CRUD operations |
| **MemoPilot [N6]** | Explicit cross-round text memory | Memory-update copilot guides player | Copilot trained with multi-turn GRPO; **player frozen** |
| **NapMem [N7]** | Linked memory pyramid from raw conversations to profiles | Learned tool-navigation policy | Memory-using agent via memory-tool RL |
| **Memory-R2 [N8]** | Accumulated and mutable memory over session stream | Coupled extractor/manager roles | Shared-parameter RL with local/global credit signals |

**No row should be interpreted as H3 or H4 merely because memory changes.** For harness H4, both the *executor parameters and executable harness* must be adapted in a coupled loop, as specified in the [harness taxonomy](../../agent-systems-engineering/taxonomy.md).

## Primary research and evaluation

### N1 — Memorilla: Latent Semantic Memory for LLMs

- **Adibvafa Fallahpour, Parsa Idehpour, Vignesh Kothapalli, Nikita Mounier, Shirley Wu, Shayan Pardis, Jure Leskovec.** **2026-10-15**, arXiv v1 preprint. [Primary paper](https://arxiv.org/abs/2610.21375). [Project](https://memory.stanford.edu/).
- **Architecture:** document encoder creates reusable embeddings; learned query-conditioned module projects information into the LLM embedding space as **16 memory tokens**. The executor LLM is frozen. A memory token is a continuous representation, **not** a textual chunk or an updated LLM weight.
- **Reported evidence:** best among tested baselines on **five of eight** benchmarks. FactKG accuracy **84.33% vs 80.43%** for full context with **108× fewer source-representation tokens**. In TextWorld, success **24.2% vs 18.6%** for the paper's full-history comparator.
- **Limits:** benchmarks where it *does not* win; reader training and ingestion cost; memory updates and temporal correction; fairness of comparisons between short latent inputs and long text; robustness when document/interaction distribution shifts. The 108× token figure is *not* a measured 108× total computational saving.
- **Classification:** primary M3, secondary [M1](../m1-memory-systems/research.md).

### N2 — Context Language Models (CLM)

- **Rulin Shao et al.** **2026-09-29**, arXiv v1 preprint. [Primary paper](https://arxiv.org/abs/2609.37725). [Code](https://github.com/facebookresearch/context-language-models).
- **Architecture:** context treated as an editable file; model can update its own working memory and can interact with multiple agent-context files.
- **Important variants:** **zero-shot CLM** already outperforms external-context-management approaches on tested tasks; **optimized natural-language instructions** change harness/prompt policy without necessarily training weights; **online RL** updates model parameters for learned management. Do not describe every CLM experiment as RL-trained.
- **Reported results:** zero-shot gains of **11.4% accuracy with 21.5% fewer FLOPs on BrowseComp-Plus** and **5% higher scores with 59% fewer FLOPs on 12h EdgeBench**, per authors. The RL variant reports **47.6% improvement of Qwen3.5-9B on BrowseComp-Plus with 12% fewer FLOPs**; custom Suffix Cache Reuse provides separate serving-system gains.
- **Limits:** different improvements come from **different mechanisms**; isolate prompt/policy learning, file management, executor model, suffix-cache serving and benchmarks. No inherent guarantee of safe, accurate long-term cross-session storage.
- **Classification:** M3 when studying trained context policy; M2/H1 when studying deterministic or zero-shot file operations.

### N3 — Just-In-Time Agent Memory with Runtime Agentic Research (JAM)

- **Bingyu Yan, Chaofan Li, Hongjin Qian, Shuqi Lu, Chaozhuo Li, Zheng Liu.** **2026-09-28**, arXiv preprint. [Primary paper](https://arxiv.org/abs/2609.34385). [Code](https://github.com/VectorSpaceLab/general-agentic-memory).
- **Mechanism:** fixed **Memorizer** saves raw sessions in a hierarchical page-store with navigating summaries; query-dependent **Researcher** calls open/search/browse iteratively, checks sufficiency and builds evidence-grounded context.
- **Training:** verified-trajectory SFT followed by **Hint-guided GRPO**, with **source recall** as core training reward; the Memorizer stays fixed while the Researcher learns.
- **Reported evidence:** across LoCoMo, LongMemEval, NarrativeQA, and HotpotQA, the paper reports gains versus AOT memory and training-based baselines. **Caveat:** some learned competitors used different 7B–14B checkpoints and Memory-R1 numbers were reported from its original paper rather than all rerun under a single backbone.
- **Interpretation:** a clear example of **trainable retrieval policy over preserved raw history**, complementary to Memorilla's learned latent projection. Success hinges on the validity of generated training supervision and total tool-call costs.
- **Classification:** M3 primary, M1 secondary.

### N4 — Memory-R1: Enhancing Large Language Model Agents to Manage and Utilize Memories via Reinforcement Learning

- **Sikuan Yan et al.** First submitted **2025-08-27**; revised **2026-01-14** (v5); arXiv preprint. [Primary paper](https://arxiv.org/abs/2508.19828).
- **Mechanism/training:** Memory Manager learns **ADD / UPDATE / DELETE / NOOP**; an Answer Agent selects/uses memories. The authors optimize both roles with **PPO and GRPO**, rather than relying only on hard-coded memory prompts.
- **Evidence:** tested on LoCoMo, Multi-Session Chat and LongMemEval with 3B–14B model sizes; authors report adaptation with just 152 QA training pairs.
- **Limits:** need checks for prompt/schema bias, data contamination, exact comparability of differently sized models, and handling contradictory or sensitive facts.
- **Importance:** baseline for trained memory operations; establishes M3 as more than just latent-memory architecture.

### N5 — Learning to Remember: End-to-End Training of Memory Agents for Long-Context Reasoning (UMA)

- **Kehao Zhang, Shangtong Gui, Sheng Yang, Wei Chen, Yang Feng.** **2026-02-13**, arXiv preprint. [Primary paper](https://arxiv.org/abs/2602.18493). [Code](https://github.com/ictnlp/unified-memory-agent).
- **Mechanism:** combines a compact core summary with a structured memory bank supporting **create/update/delete/reorganize**; same learned policy handles stream memory updates and later QA.
- **Training/evaluation:** end-to-end RL; evaluates **13 datasets**, including diagnostic **Ledger-QA**, where the answer depends on applying evolving state updates instead of locating one static passage.
- **Interpretation:** directly targets *state tracking* and learned consolidation, a different goal from Memorilla's memory-token compression.
- **Limit:** verify that improvement persists under changing/ambiguous facts, shifted schemas, unseen tasks and matched update budgets. Continual consolidation is not automatically reliable: see counterevidence from [M2](../m2-memory-lifecycle/research.md).

### N6 — From Player to Master: ... Reinforcement Learning over Memory (MemoPilot)

- **Yishuo Cai et al.** **2026-06-07**; arXiv preprint, **accepted ICML 2026**. [Primary paper](https://arxiv.org/abs/2606.08656).
- **Mechanism:** a plug-in memory copilot learns **what to update across turns**, optimizing a **frozen** decision-making player through a multi-turn GRPO formulation and turn-wise reward/advantage design.
- **Reported evidence:** author tests on repeated Rock-Paper-Scissors and limit Texas Hold'em, with top reported Elo scores **1,590 and 1,762** respectively under their evaluation.
- **Limits:** restricted strategic game dynamics; evidence does not show success on free-form enterprise assistant memories or source-grounded multi-session fact recall. Separate training cost from test-time player improvement.
- **Interpretation:** strong conceptual demonstration of trained memory updates without adapting executor weights.

### N7 — From Passive Retrieval to Active Memory Navigation (NapMem)

- **Yue Xu et al.** **2026-07-07**, arXiv preprint. [Primary paper](https://arxiv.org/abs/2607.05794).
- **Mechanism:** linked multi-granularity user-memory pyramid (raw messages, records, topics, profile) and learned **memory-tool RL navigation**; agent chooses which level to inspect.
- **Evidence:** PersonaMem-v2, LongMemEval and LoCoMo; paper investigates non-memory capability retention and tool-navigation ablations.
- **Limits:** personalization-specific workload, policy safety and leakage; multigranular memory can increase maintenance and scope-control complexity.

### N8 — Memory-R2: Fair Credit Assignment for Long-Horizon Memory-Augmented LLM Agents

- **Sikuan Yan et al.** **2026-05-20**, arXiv preprint. [Primary paper](https://arxiv.org/abs/2605.21768).
- **Core problem:** different RL rollout branches mutate their own memory states, invalidating a simple assumption behind group-relative trajectory comparisons.
- **Proposed training:** **LoGo-GRPO** combines global trajectory reward with local rerollouts launched from the **same intermediate memory state**. Fact extractor and manager share learned parameters through roles; horizon curriculum **8 → 16 → 32 sessions**.
- **Interpretation:** precise algorithmic issue for long-horizon memory policy learning, beyond merely attaching GRPO to any agent. Measure extra sampling expense, matched trajectory budgets, stability and generalization to longer unseen horizons.

### N9 — All-in-One Memory: Process-Rewarded Reinforcement Learning for Memory Agents (AiO-Mem)

- **Bingbing Wang, Jingjie Lin, Weiming Qiao, Jing Li, Ruifeng Xu.** **2026-10-15**, revised **2026-10-16** (v2); arXiv preprint. [Primary paper](https://arxiv.org/abs/2610.21147).
- **Mechanism:** source-traceable event memory; a unified actor handles search, write and stopping decisions with shared evidence state. A separately trained *process reward model* uses evidence-grounded SFT and trajectory-level DPO; PA-DAPO optimizes the memory policy with aggregated process rewards.
- **Reported evidence:** author-reported gains over static and RL-based agents on long-context memory tests.
- **Limits:** needs ablation showing added value of process reward vs larger search/rollout budgets, reward-model leakage, and honest accounting of grader training cost. Novel name alone is not proof of a fundamentally new RL family.

## Experimental contradictions / methodological cautions

| Hypothesis | Supporting studies | Qualification / counter-test |
| --- | --- | --- |
| Learned memory improves frozen executors | Memorilla, JAM, MemoPilot | Backbone, stored evidence and token/compute budgets differ; verify same-executor controlled baselines |
| Learned write/update policies beat heuristics | Memory-R1, UMA, MemoPilot | [M2](../m2-memory-lifecycle/research.md) reports forced consolidation can harm performance; compare episodic-only and revision-gated alternatives |
| RL trajectory reward is sufficient for memory | Memory-R1, UMA | Memory-R2 argues rollout state mismatch makes naive group-relative credit biased; audit rollout branches and state resets |
| Shorter context reduces *overall* system cost | Memorilla, CLM, JAM | Must include indexing, encoding, tool calls, policy training and serving optimizations separately |

## Monitoring priorities

- Track **precise trained parameters**, not labels: executor vs memory controller vs trainable encoder/latent adapter; where weight updates happen and what remains external mutable data.
- For RL, record **MDP/observation/action/state, reward shaping, credit assignment, eligibility of memory states for comparison, rollout costs, held-out policy transfer**.
- For neural memories, test retention over restarts, additions/corrections, gradient interference, OOD retrieval and whether embeddings encode outdated/private facts.
- Cross-link M1 storage, M2 retention/deletion and M4 experience-to-skill; do not duplicate their detailed findings.
- Benchmark matched no-memory, simple RAG, full-history, heuristic write/read, non-training memory and trainable memory systems. Distinguish measured answer improvements from robust lifelong adaptation.

**Evidence grade:** MemoPilot states ICML 2026 acceptance; other cited studies are arXiv preprints in the currently verified metadata. All numbers are author-reported; no independent rerun was conducted in this review. Maintain this file as a corrected evidence map, never append unfiltered weekly lists.
