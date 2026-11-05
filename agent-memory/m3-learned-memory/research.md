# M3 — Learned Memory & Model–Memory Co-Adaptation — Living Research State

_Reviewed: 2026-11-05. Updated learned memory policies, co-adaptation, neural-state boundaries and Memorilla v2, preserving prior N1–N9 evidence. This scheduler owns **only this file**; parent [research.md](../research.md), [taxonomy](../taxonomy.md), [study guide](../study.md) and sibling files remain read-only._

## Current synthesis

1. **“Learned memory” does not imply updating the backbone that answers requests.** Memorilla trains a reader projecting external document embeddings into a frozen executor's input space; JAM trains a separate Researcher; MemoPilot trains a memory-update copilot for a frozen player. CLM, Memory-R1 and UMA also investigate learned context/memory operations, with different trainable components. These are not interchangeable architectures. [N1–N6]
2. **There are at least three different learned decisions:** *(a)* compress/represent stored evidence into tokens (Memorilla), *(b)* navigate a source-preserving external store and choose query-time evidence (JAM, NapMem), *(c)* decide how to create/update/delete memory over a stream (Memory-R1, UMA, MemoPilot, Memory-R2). Judge their contribution against the matching non-learning baseline. [N1, N3–N7]
3. **Training a memory agent is a long-horizon RL problem with nonstationary state caused by its own actions.** Memory-R2 observes that different rollout branches no longer share an equivalent memory history; its local rerollouts offer a principled response to unfair group-relative credit assignment. It is still an author-reported preprint result. [N8]
4. **Model-controlled files are not necessarily trained policies.** CLM includes zero-shot context-file use, instruction search and parametric RL variants. Only the learned variants are strong M3 exemplars; plain file management belongs mainly to M2/H1. [N2]
5. **Learned memory assembly is not memory writing.** MemComposer [N10] trains a separate RL composer to choose and order fixed text episodes for a frozen executor. Its gains against several strong learned-memory baselines are not statistically distinguishable in all tested settings.
6. **Parameter-level co-adaptation is demonstrable but uncommon.** ElasticMem [N11] jointly optimizes executor attention LoRA adapters, a latent projector, and a memory-budget policy while keeping its source encoder and external bank frozen. This contrasts with Memorilla's frozen answering backbone and MemComposer's frozen executor.
7. **Neural state is not automatically persistent agent memory.** LSTMem [N12] updates a compact recurrent state over a history but resets per independent history and still retains KV history during evaluation. Titans [boundary] studies test-time fast weights within sequences, not an auditable cross-restart agent memory.
8. **The strongest remaining uncertainty is cost-matched generalization.** Token reductions can hide encoder passes, repeated tool calls, attention passes, index/cache storage, LoRA or RL training, simulated executor rollouts and serving optimizations. Most benchmark comparisons are author-reported and do not establish safe lifelong updates, deletion, restart persistence or matched total FLOPs.

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
| **MemComposer [N10]** | Fixed external bank of textual experiences | Composer selects/counts/orders episodes | Separate composer LoRA trained with GRPO-like RL; **executor and bank frozen** |
| **ElasticMem [N11]** | Fixed offline embedding keys and cached latent document states | Learned retrieval trigger, budget and projection | **Executor attention LoRA + budget policy + projector jointly trained**; base and encoder frozen |
| **LSTMem [N12]** | Per-layer recurrent cell/hidden matrices within one history | Gated adapter reads/writes neural state | Adapter weights supervised-trained; **backbone frozen**; recurrent state changes at inference |
| **Titans [boundary]** | Sequence-time neural fast weights and input-independent persistent tokens | Surprise-gradient state updater | Fast weights update during sequence; persistent tokens are fixed at inference |

**No row should be interpreted as H3 or H4 merely because memory changes.** For harness H4, both the *executor parameters and executable harness* must be adapted in a coupled loop, as specified in the [harness taxonomy](../../agent-systems-engineering/taxonomy.md).

## Primary research and evaluation

### N1 — Memorilla: Latent Semantic Memory for LLMs

- **Adibvafa Fallahpour, Parsa Idehpour, Vignesh Kothapalli, Nikita Mounier, Shirley Wu, Shayan Pardis, Jure Leskovec.** Submitted **2026-10-15**, revised **2026-10-27 (v2)**; arXiv preprint. [Primary paper](https://arxiv.org/abs/2610.21375). [v2 methods](https://arxiv.org/html/2610.21375v2). [Project](https://memory.stanford.edu/).
- **Architecture and trained components:** frozen Qwen3-Embedding-4B encodes documents/query into reusable embeddings; a learned query-conditioned slot/cross-attention/projector module (reported **194,123,776 trainable parameters**) converts them to **16 continuous memory tokens** for a **frozen Qwen3-8B executor**. The module is trained using downstream answer-token objectives. Memory tokens are not textual chunks or changes to backbone weights. The TextWorld GRPO variant uses **eight** tokens but also retains full trajectory text under an evaluation context limit, so it does not prove independent durable lifelong memory.
- **Reported evidence:** best among tested baselines on **five of eight** benchmarks. FactKG accuracy **84.33% vs 80.43%** for full context with **108× fewer source-representation tokens**. In TextWorld, success **24.2% vs 18.6%** for the paper's full-history comparator.
- **v2 cost claim and confounders:** at a tested 512-document setup, authors report **0.74 s vs 19.93 s** against full context on an H100. The long-context serving baseline uses YaRN and an unpaged HF inference path; this is **not** a general 27× deployment speedup. The **108× figure** concerns *source-context token reduction*, not full system FLOPs. Training and document encoding must be included in total costs; some no-document LoRA comparisons use a different decoder. Test dynamic updates, correction, deletion, restart persistence and comparisons to equally supervised learned retrieval.
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

### N10 — MemComposer: Learning to Assemble Memories for LLM Agents via Reinforcement Learning

- **Yixuan Xie, Xueqiang Xu, Xinyi Fan et al.** Submitted **2026-10-15**. **Status:** arXiv preprint. [Primary](https://arxiv.org/abs/2610.21091), [full methods](https://arxiv.org/html/2610.21091).
- **Substrate and read/write boundary:** a fixed bank of curated textual episodes from earlier agents. A separate **composer** chooses the **number and order** of episodes assembled into the prompt of a **frozen executor**. The composer does not create, delete, or retrain the experience bank in this setup.
- **What is trained:** Qwen3-4B-Instruct or Ministral-3-8B-Instruct composer with **rank-64 LoRA**, optimized by ExPO/GRPO-style group comparisons over repeated success of frozen executor rollouts, penalizing unnecessarily large/invalid compositions. Frozen: executor and bank.
- **Author-reported evidence:** on portions of tau-squared-bench, Qwen executor **pass^1 50.8 vs 45.0** strongest tested comparator; Ministral **63.5 vs 58.9**. Mean memory-context tokens decline **16.9%** and **4.9%**, respectively. The paper's **pass^4 requires success in all four independent runs** (not pass@4). Qwen gap versus MemRL **+6.3 percentage points, 95% paired-bootstrap CI [-0.4,12.7]**; Ministral **+4.6, CI [-1.0,10.4]**. Thus superiority over that strong baseline is **uncertain** in these tests.
- **Costs and limits:** author reports Qwen **16.3–81.7 H100 GPU-hours per domain**, excluding relevant external executor/user-simulator API charges; repeated executor simulations dominate training runtime. Some domains have only 20–40 test tasks, without robust multiplicity correction; transfer to unfamiliar banks, shifted settings or changed executors is limited. Reduced prompt tokens are not a substitute for full training and serving cost.
- **Assessment:** strong example of **learned read/assembly with a frozen executor**, not demonstrated neural/external memory co-training. **M3 primary, M4 secondary** for the experiential substrate.

### N11 — ElasticMem: Latent Memory as a Learnable Resource for LLM Agents

- **Tao Feng, Chongrui Ye, Tianyang Luo et al.** Submitted **2026-05-29**. **Status:** arXiv preprint; older paper newly added because it fills a material co-adaptation gap. [Primary](https://arxiv.org/abs/2605.30690), [full methods](https://arxiv.org/html/2605.30690).
- **Substrate:** *offline fixed* bank of chunks represented by keys (encoder last hidden state) and latent-content caches (often last 20 hidden states); a **frozen encoder** constructs the bank. Executor emits a memory-control token and hidden query, retrieves chunks, chooses **per-chunk latent budget**, and projects retrieved states as soft tokens.
- **What is trained:** **reasoner attention q/k/v/o LoRA adapters (rank 16)**, latent projector, and budget-policy module **jointly with GRPO**. **Frozen:** base reasoner weights outside LoRA, source encoder, and offline bank. This is authentic **executor-adapter plus memory interface co-adaptation**, not automatically harness H3/H4.
- **Reported evidence:** Qwen2.5-3B/7B experiments claim **26.2%/24.6% relative weighted QA accuracy** and **66.3%/27.2% relative ALFWorld success** versus best studied baselines. **Relative improvements are not percentage-point increases.**
- **Limitations:** shared backbone scale/source evidence does not imply equal training objectives, FLOPs, inference overhead or labels. ALFWorld optimization uses expert-action agreement as training feedback while task success is evaluated; tasks may be retrained separately. Dynamic inserts, contradictory facts, safe deletion, and long-term restart consistency remain untested.
- **Next ablations:** same LoRA executor and memory bank with fixed retrieval/budget, projector-only, budget-only and jointly trained modules at matched training+inference compute. **M3 primary; M1 secondary for latent substrate.**

### N12 — LSTMem: Hierarchical Long Short-Term Online Memory for Large Language Models

- **Xianglong Shi, Ruijie Yang, Sirui Zhao et al.** Submitted **2026-09-27**, revised **2026-10-19 (v2)**. **Status:** arXiv preprint. [Primary](https://arxiv.org/abs/2609.33268), [code](https://github.com/Longchentong/LSTMem).
- **Substrate:** per-layer **gated recurrent cell and hidden-state matrices**, with cross-layer state transfer and state correction at block boundaries. Main Qwen3-4B configuration reports **36 KiB BF16 C/D state** *per history*, independent of history length. Not an external document database.
- **What is trained:** auxiliary memory adapter via **supervised answer-token cross entropy**; backbone weights **frozen**. During inference the recurrent **state changes**, but the adapter/LLM weights do not. Do not mistake gradients over a temporary state for training of executor parameters.
- **Reported evidence:** Qwen3-4B LoCoMo F1 **53.60 vs 49.12** for one online-memory competitor; aggregate history inference **5.9–6.5% slower** than delta-Mem across tested 8K–128K settings. Some competitors' results are imported from earlier studies.
- **Critical qualification:** state **resets for each independent conversation/history**, and evaluation retains the **historical KV cache**; several questions about one history do not create a persistent multi-session state. Thus a 36 KiB recurrent state is not evidence of a complete 36 KiB durable lifelong agent memory. No direct safe removal or restart isolation is established.
- **Classification:** boundary M3 for learned neural-memory adapter; do not treat it as equivalent to persistent inter-task M1 storage.

### Additional boundaries — Titans and MemoProAgent

- **Titans: Learning to Memorize at Test Time** (Ali Behrouz, Peilin Zhong, Vahab Mirrokni; first posted **2024-12-31**), [primary](https://arxiv.org/abs/2501.00663): sequence-time fast neural weights updated through surprise/gradient dynamics, plus separately learned input-independent persistent tokens fixed at inference. The term *persistent* is not evidence of autonomous-agent memory surviving a process restart, with source provenance, retrieval ACLs and controlled deletion.
- **MemoProAgent: Toward Memory-Driven Proactive Assistance** (Shenglai Zeng et al.; revised **2026-10-21**), [primary](https://arxiv.org/abs/2610.22671): zero-shot graph memory and firing control are separable from GRPO **post-training of the proactive generator**. Reported strict proactive scores **37.5% zero-shot vs 42.4% with generator-policy RL** on a tested Qwen3.8-27B setting. The trained component is the **generator decision policy**, not established as a learned write/read/forget controller; track as **boundary evidence**, not a primary M3 memory-policy paper.

## Experimental contradictions / methodological cautions

| Hypothesis | Supporting studies | Qualification / counter-test |
| --- | --- | --- |
| Learned memory improves frozen executors | Memorilla, JAM, MemoPilot | Backbone, stored evidence and token/compute budgets differ; verify same-executor controlled baselines |
| Learned write/update policies beat heuristics | Memory-R1, UMA, MemoPilot | [M2](../m2-memory-lifecycle/research.md) reports forced consolidation can harm performance; compare episodic-only and revision-gated alternatives |
| RL trajectory reward is sufficient for memory | Memory-R1, UMA | Memory-R2 argues rollout state mismatch makes naive group-relative credit biased; audit rollout branches and state resets |
| Shorter context reduces *overall* system cost | Memorilla, CLM, JAM, ElasticMem | Include encoding, latent-cache storage, LoRA/GRPO training, repeated rollouts, tool/API cost and serving optimizations |
| Learned selection improves over strong baselines | MemComposer [N10] | Confidence intervals against some competitors cross zero; training involves costly simulated executor rollouts |
| Executor weights always stay frozen | Memorilla, MemComposer | **False for ElasticMem [N11]: reasoner attention LoRA is trained jointly with memory policy/projector** |
| Neural recurrent state proves lifelong persistence | Titans and LSTMem show dynamic in-sequence memory | LSTMem resets per independent history and retains historical KV; neither verifies cross-restart agent memory |

## Monitoring priorities

- Track **precise trained parameters**, not labels: executor vs memory controller vs trainable encoder/latent adapter; where weight updates happen and what remains external mutable data.
- For RL, record **MDP/observation/action/state, reward shaping, credit assignment, eligibility of memory states for comparison, rollout costs, held-out policy transfer**.
- For neural memories, test retention over restarts, additions/corrections, gradient interference, OOD retrieval and whether embeddings encode outdated/private facts. **Explicitly record state resets and whether source historical KV remains resident.**
- **Causal comparison:** same answering backbone, source corpus, candidate list and objective across strong raw-history RAG/full history, zero-shot or rule-based selection, MemComposer learned assembly, frozen-backbone Memorilla, and ElasticMem-style trainable executor adapters. Equate supervision, train FLOPs and full read/write serving budgets.
- **Co-adaptation control:** under identical LoRA resources test projector only, memory-budget policy only, fixed retrieval, and joint LoRA/projector/policy learning; require OOD task transfer and dynamic writes/corrections.
- Cross-link M1 storage, M2 retention/deletion and M4 experience-to-skill; do not duplicate their detailed findings.
- Benchmark matched no-memory, simple RAG, full-history, heuristic write/read, non-training memory and trainable memory systems. Distinguish measured answer improvements from robust lifelong adaptation.

**Evidence grade / review continuity:** baseline N1–N9 from 2026-10-20 retained; revised Memorilla v2 metadata/limitations and added N10–N12 with Titans/MemoProAgent boundary checks on 2026-11-05. MemoPilot reports ICML 2026 acceptance; N10–N12 are author-reported preprints. No independent reproduction or benchmark was run. Maintain this document as living evidence, not an append-only event log.
