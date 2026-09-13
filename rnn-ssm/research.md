# RNN and SSM Research Memory

_Last updated: 2026-09-12_

This is a curated living document for modern recurrent sequence models, State Space Models (SSMs), linear attention, recurrent-depth models, and recurrent/attention hybrids in NLP and LLMs. It is organized by the current conceptual structure of the field rather than by weekly scheduler runs.

## Current assessment

The field is no longer well described as "RNN vs Transformer." The active design space now contains at least two distinct kinds of recurrence:

1. **Token-time recurrence**: a fixed-size state is updated while moving through sequence positions. This includes SSMs, linear attention in recurrent form, RWKV, xLSTM, Hawk/Griffin, and related hybrids.
2. **Loop-time / recurrent-depth recurrence**: the same or shared computation block is repeatedly applied to an internal representation to spend more computation on the same input. This includes recurrent-depth/looped Transformers, RecurTrace, and the new Looped Flows line.

For token-time models, the most useful conceptual decomposition is increasingly:

- **write/update**: what new information enters the state;
- **erase/forget**: what old information is attenuated or overwritten;
- **read/query**: how the compressed state is interrogated;
- **uncertainty/confidence**: how strongly the model should trust the existing state;
- **serving/reconstruction**: how recurrent states can be cached, reused, quantized, or reconstructed in production.

Hybrid architectures remain especially credible. Recent causal evidence suggests that full attention and recurrent state can learn different memory roles rather than being redundant approximations of each other: attention is strong at exact addressable retrieval/bindings, while recurrence can carry compressed semantic, stylistic, or behavioral context.

---

## Material changes since the previous report

### 1. Thinking with Looped Flows

**Status:** NEW — conceptually important; arXiv v1, 2026-09-10  
**Authors:** Ayhan Suleymanzade, Chanhyuk Lee, Floor Eijkelboom, Nicholas M. Boffi, İsmail İlkan Ceylan, Jinwoo Kim  
**Affiliations reported across the paper/project:** EPFL, KAIST, University of Amsterdam, Carnegie Mellon University, TU Wien, AITHYRA, University of Oxford  
**Primary source:** https://arxiv.org/abs/2609.11801

#### Contribution

Looped models/recurrent-depth models repeatedly update a hidden state, but full BPTT through many loop iterations is expensive and unstable. Practical methods therefore often stop gradients between iterations or backpropagate through only a short recurrence, which weakens temporal credit assignment: early loop states are not directly trained to support much later computation.

Looped Flows changes the training problem rather than merely changing the recurrent block. It trains a **stateful denoiser** over a sequence of progressively lower-noise local denoising objectives. Consecutive objectives share the same noise/target pair, creating temporal association across loop iterations even when gradients do not propagate through the full recurrence. At inference, the model couples the recurrent hidden state to a probability-flow trajectory and can spend more compute by using a finer integration grid.

A simplified view is:

- looped model: `z_i = f(z_{i-1}, c)`;
- looped flow: a stateful denoiser jointly updates a prediction/flow state and recurrent state, while each iteration solves a locally supervised denoising problem whose difficulty decreases over time.

This is **not the same mechanism as RecurTrace**. RecurTrace improves recurrent depth by making previous loop states explicitly addressable with Loop Memory Attention and by learning when to halt. Looped Flows instead changes how recurrence is trained and how inference-time computation is parameterized, using flow/denoising structure to make local training signals temporally coherent.

#### Evidence

The paper reports results across six reasoning benchmarks, outperforming prior looped-model baselines on five and remaining competitive on the sixth. With the same architecture as TRM, it reports:

- ARC-AGI-1: **58.8%** vs **44.6%** for TRM;
- ARC-AGI-2: **12.2%** vs **7.8%** for TRM.

It also reports stable improvement as inference-time computation increases and supports multiple valid solutions by sampling different initial noise states.

#### Limitations / evidence quality

- v1 preprint; no peer-reviewed status yet.
- Evidence is on reasoning/problem-solving benchmarks, not general next-token pretraining at LLM scale.
- The method adds a flow/denoising formulation and numerical integration machinery; it is not yet clear whether this scales cleanly to large autoregressive language models.
- It strengthens the **recurrent-depth** branch, not the token-time SSM/linear-attention branch.

#### Assessment

**Conceptual change. High priority.** The most important new result this run. It reframes recurrent-depth training as a temporal credit-assignment problem and uses local denoising/flow objectives to solve it without full long-horizon BPTT.

---

### 2. Learning Length-Extrapolatable Recurrent Models

**Status:** NEW — conceptually important training result; arXiv v1, 2026-09-08  
**Author:** Hanwen Jiang  
**Institution:** Adobe Research  
**Primary source:** https://arxiv.org/abs/2609.09157

#### Contribution

The paper argues that the standard diagnosis "recurrent models fail at long horizons because gradients vanish/explode" is incomplete. Dense per-token losses can still train a shared recurrent rule even when long-path gradients decay severely. The more precise object is **state credit**: the signal by which future losses assign responsibility to earlier recurrent states before parameter gradients are formed.

It proposes **Credit Stabilization through Time (CST)**. During backward propagation, CST rescales the state-credit signal locally to stabilize its norm while preserving the direction/component being corrected; the forward recurrence is unchanged.

This matters because length extrapolation is one of the central advertised advantages of recurrent/SSM-style language models: a fixed-size recurrent state does not intrinsically require positional extrapolation or a growing KV cache, but the recurrent rule may still fail when executed far beyond the training horizon.

#### Evidence

The paper reports improvements on controlled synthetic tasks and real-data settings, with gains observed at evaluation lengths up to **128x the training length**.

#### Limitations / evidence quality

- v1 preprint and currently a single-author result.
- The strongest headline is length extrapolation, not broad language-model quality.
- The result needs replication across larger model families and training scales before treating CST as a general replacement for ordinary BPTT recipes.
- The intervention is in the backward pass; it does not directly improve state capacity, associative recall, or retrieval precision.

#### Assessment

**Conceptually important.** This is a useful refinement of the classical vanishing/exploding-gradient story. Track it as a training/credit-assignment line alongside architectural work such as Mamba/GDN and recurrent-depth work such as RecurTrace/Looped Flows.

---

### 3. Global Divergence, Local Convergence: Representation Geometry in SSMs and Transformers

**Status:** NEW — rigorous comparative analysis; arXiv v1, 2026-09-08  
**Authors:** Amit Ben-Artzy, Roy Schwartz  
**Primary source:** https://arxiv.org/abs/2609.08692

#### Contribution

This work compares internal representation geometry in Transformers, SSMs, and hybrid models rather than comparing only perplexity or throughput.

Main findings reported by the authors:

- SSM representations distribute variance/information more evenly across dimensions.
- Transformer representations are much more dominated by a single principal direction.
- In hybrid models, attention layers progressively push representations toward this more anisotropic/skewed geometry.
- Despite different global geometry, effective representational capacity is reported to be closely matched.
- Rank-constrained probes suggest concepts occupy subspaces of similar dimensionality in both families.
- At the local level (topic manifolds and nearest-neighbor token neighborhoods), SSM and Transformer representations are highly aligned.

#### Relation to prior work

This complements architectural duality results such as Mamba-2/SSD. Mamba-2 shows algebraic/computational relationships between SSMs and attention. This paper asks whether differently implemented models also converge toward similar semantic structures internally.

It is also relevant to the broader question of whether SSMs are merely compressed approximations of Transformer representations. The reported answer is more nuanced: **global latent geometry differs, but local semantic organization can converge strongly.**

#### Limitations / evidence quality

- Analysis/probing work rather than a causal architectural intervention.
- Conclusions depend on the model families, layers, and datasets studied; geometric results should not automatically be generalized to every SSM or hybrid.
- Similar probe dimensionality does not prove identical functional mechanisms.

#### Assessment

**Important evaluation/mechanistic result, not an architectural advance.** Particularly useful for connecting architecture research to representation geometry.

---

### 4. Looped GPT-BERT: Trading Parameters for Computation in Small Language Modeling

**Status:** NEW — supporting evidence for recurrent depth; arXiv v1, 2026-09-09  
**Authors:** Tingshuo Fan, Hongtao Mu, Tianyu Zhou, Hansen Liu, Tao Ji  
**Primary source:** https://arxiv.org/abs/2609.09691

#### Contribution

Tests depth-wise parameter sharing in the BabyLM 2026 Strict-small regime. The final model uses **4 physical layers traversed 12 times** (`4x12`) and 12.18M parameters, explicitly trading stored parameters for recurrent computation.

This provides a clean small-model demonstration of the recurrent-depth thesis: a small set of physical layers can be reused to obtain a larger effective computational depth.

#### Evidence

- Training corpus: 7.48M English words.
- BabyLM 2026 leaderboard: Overall Average **35.42**, NLP Average **48.48**.
- Comparable results to public 10M Strict-small GPT-2/GPT-BERT baselines on selected metrics such as BLiMP and GLUE with fewer parameters.

The paper's own ablations also reveal an important counterpoint: additional loops help some tasks, while having very few distinct physical layers can constrain representational diversity and hurt others.

#### Limitations / evidence quality

- Very small model and very small-data regime.
- Not evidence that the same parameter/computation tradeoff holds at multi-billion-parameter LLM scale.
- Gains are mixed across tasks rather than uniformly dominant.

#### Assessment

**Incremental/supporting evidence.** Keep as a clean empirical example of recurrent depth and parameter sharing, but below RecurTrace and Looped Flows in priority.

---

## Active token-time architecture map

### Mamba / SSM line

**S4 -> LRU -> Mamba -> Mamba-2 -> Mamba-3**

- **Mamba** — selective SSMs made state updates input-dependent and brought SSMs into serious language-model competition.  
  Primary: https://arxiv.org/abs/2312.00752
- **Mamba-2 / State Space Duality** — unifies structured SSM computation with attention-like matrix views and improves hardware efficiency.  
  Primary: https://proceedings.mlr.press/v235/dao24a.html
- **Mamba-3** — ICLR 2026 Oral; improves state tracking, discretization/expressivity, MIMO structure, and real decoding efficiency.  
  Primary: https://arxiv.org/abs/2603.15569

**Current status:** still central and **unchanged this run**. No new Mamba paper/revision found that supersedes Mamba-3.

### Linear attention / associative-memory line

**Linear attention -> GLA -> DeltaNet -> Gated DeltaNet -> KDA/Kimi Linear -> Gated DeltaNet-2 / Kalman Delta Networks**

- **Gated DeltaNet** combines global forgetting/gating with delta-rule associative-memory updates.  
  Primary: https://openreview.net/forum?id=r8H7xhYPwz
- **Kimi Linear / KDA** adds more granular gating and demonstrates a large hybrid architecture.  
  Primary: https://arxiv.org/abs/2510.26692
- **Gated DeltaNet-2** decouples erase and write operations.  
  Primary: https://arxiv.org/abs/2605.22791
- **Kalman Delta Networks (KDN)** adds explicit uncertainty/confidence tracking to associative memory, interpreting ordinary delta-style updates as an approximation to Kalman filtering when covariance tracking is collapsed.  
  Primary: https://arxiv.org/abs/2609.07816

**Current assessment:** KDN remains the most interesting recent conceptual extension of the write/erase line. **Still relevant; no substantive revision found this run.** The open question is whether uncertainty-aware updates retain their advantage at much larger LLM scale and in hybrid models.

### RWKV

- **RWKV original** establishes train-parallel / infer-recurrent language modeling.  
  Primary: https://arxiv.org/abs/2305.13048
- **RWKV-7 Goose** adds more expressive dynamic-state updates, vector-valued gates, and stronger state-tracking theory.  
  Primary: https://arxiv.org/abs/2503.14456

**Current status:** still relevant; **no material update this run**.

### xLSTM

- **xLSTM** modernizes the LSTM family with exponential gating and matrix memory.  
  Primary: https://arxiv.org/abs/2405.04517
- **xLSTM 7B** demonstrates a recurrent LM at multi-billion scale.  
  Primary: https://arxiv.org/abs/2503.13427
- **xLSTM Scaling Laws** studies scaling behavior and compute/quality tradeoffs.  
  Primary: https://openreview.net/forum?id=bpbU549sSg

**Current status:** still relevant; **no new result found this run that changes the assessment**.

### Griffin / RecurrentGemma and hybrids

- **Griffin/Hawk**: gated linear recurrence plus local attention.  
  Primary: https://arxiv.org/abs/2402.19427
- **RecurrentGemma**: open scaled implementation of the Griffin line.  
  Primary: https://arxiv.org/abs/2404.07839

**Current status:** conceptually important as early evidence that recurrence and local attention are complementary. No new Griffin/RecurrentGemma research update found this run.

---

## Active recurrent-depth / loop-time line

### RecurTrace — still relevant

**Primary:** https://arxiv.org/abs/2609.03379  
**Status:** arXiv preprint, 2026-09-03

RecurTrace adds **Loop Memory Attention**, allowing each looped layer to access its own hidden states from previous recurrent-depth iterations, plus a learned halting mechanism. This changes a Markov-like loop `z_r = F(z_{r-1})` into a recurrence with an explicitly addressable short history across loop-time.

**Assessment:** still relevant and now **strengthened by the appearance of Looped Flows**. Together they suggest recurrent depth has split into multiple subproblems:

- memory/access across loops (RecurTrace),
- adaptive halting/compute allocation (RecurTrace),
- temporal credit assignment/training stability (Looped Flows),
- parameter-vs-compute tradeoff in small LMs (Looped GPT-BERT).

### Thinking with Looped Flows — new leader in training methodology

See the new-item section above. This should now be treated as one of the highest-priority recurrent-depth papers to follow.

### Looped GPT-BERT — supporting evidence

Useful for parameter sharing and small-model data efficiency, but not yet strong evidence for frontier LLMs.

---

## Recent evidence carried forward from previous reports

### What Attention Recalls and Recurrence Controls in Hybrid Language Models

**Primary:** https://arxiv.org/abs/2609.04434  
**Status:** arXiv preprint, 2026-09-03  
**Assessment:** **still relevant; strengthened conceptually by newer hybrid/geometry evidence.**

Causal cache/state interventions in Qwen3.5-4B and Falcon-H1-3B suggest a functional division:

- attention/KV state preserves exact, addressable item retrieval and bindings;
- recurrent state preserves more compressed semantic/behavioral context such as language/persona/style.

This is currently one of the strongest arguments for viewing hybrids as **complementary memory systems**, not merely compromises between quality and efficiency.

### Why Gated DeltaNet Survives 4-Bit Quantization

**Primary:** https://arxiv.org/abs/2609.04098  
**Status:** arXiv preprint, 2026-09-03  
**Assessment:** **still relevant, practical evidence.**

The main mechanistic result is that GDN state error does not simply accumulate indefinitely under 4-bit quantization; delta-rule updates can overwrite/correct parts of recurrent-state error as new associations arrive. This suggests some recurrent associative-memory rules are partially self-correcting numerically.

### Modern Transformers Are Implicit Hybrids

**Primary:** https://arxiv.org/abs/2609.02986  
**Status:** arXiv preprint, 2026-09-02  
**Assessment:** **still promising but not yet strongly validated at frontier scale.**

Proposes head-wise rather than layer-wise assignment of full vs linear attention based on different functional roles. It aligns with evidence that exact retrieval and compressed recurrent memory benefit from different mechanisms.

### Curvature-Conditioned Query (CCQ)

**Primary:** https://arxiv.org/abs/2606.01294  
**Status:** EMNLP 2026 camera-ready revision reported 2026-08-30  
**Assessment:** **still relevant.**

Important because it shifts focus from write/erase dynamics to **read dynamics**: how a query interrogates an already-compressed linear-attention state. This completes the useful conceptual decomposition of recurrent memory into write, erase, read, uncertainty, and serving.

### Tail-Replay

**Primary:** https://arxiv.org/abs/2608.30310  
**Status:** arXiv preprint, 2026-08-31  
**Assessment:** **still relevant as serving/system evidence.**

Treats gated linear-attention state as structured lossy compression and reconstructs matched-prefix state by replaying only a recent suffix. Useful evidence that hybrid recurrent models are mature enough to create distinct prefix-caching/serving problems.

---

## New peripheral/watchlist items

### RiLM: Parameter-Efficient Language Modeling via Geodesic Decoding

**Primary:** https://arxiv.org/abs/2609.10305  
**Date/status:** 2026-09-09, arXiv v1

Uses a recurrent language-model trajectory on a Riemannian manifold and removes the conventional output projection, decoding next-token probabilities via geodesic distance to vocabulary embeddings. Hyperbolic variants outperform matched tiny LSTM/Transformer/SSM controls in the reported sub-million-parameter experiments.

**Assessment:** **deprioritized/watchlist.** The geometry is interesting, but the evidence is intentionally scoped to tiny models, small vocabularies, and controlled corpora. It is not yet evidence about modern LLM-scale recurrent architectures.

### ConvMem: Convolutional Memory for Long-Context Reasoning

**Primary:** https://arxiv.org/abs/2609.10441  
**Date/status:** 2026-09-09, arXiv v1

Replaces a sequential chunk-memory chain with hierarchical convolution/tree-style summarization using an LLM as the kernel, reducing the reasoning path from linear to logarithmic depth and enabling parallelism.

**Assessment:** **peripheral to this stream.** Relevant to long-context memory and recurrence at the system level, but it is primarily a context-processing/orchestration method rather than a new recurrent sequence-model architecture.

---

## Main open questions

1. **State capacity vs exact recall:** how much exact binding/retrieval can fixed-state models recover without reintroducing an explicit addressable memory?
2. **Functional specialization in hybrids:** can we learn routing between full attention and recurrent/linear memory rather than fixing layer/head patterns manually?
3. **Uncertainty-aware memory:** does KDN-style covariance/confidence tracking remain useful at 7B+ and frontier-scale training?
4. **Read dynamics:** can CCQ-like query conditioning close a meaningful part of the recall gap without increasing state size?
5. **Length extrapolation:** does state-credit stabilization remain effective for large language models trained on real long-context corpora?
6. **Recurrent depth scaling:** do RecurTrace/Looped Flows benefits survive at general-purpose LLM scale, or are they primarily reasoning-specialist effects?
7. **Loop-time memory:** how should models choose what intermediate computations to preserve, compress, and revisit across recurrent-depth iterations?
8. **Token-time vs loop-time recurrence:** can the two be composed effectively—e.g. a recurrent/SSM token mixer whose internal computation itself uses adaptive recurrent depth?
9. **Hardware reality:** which theoretically linear/recurrent mechanisms deliver end-to-end latency and memory advantages on production accelerators after kernel, batching, and prefix-cache constraints are included?
10. **Geometry and function:** do the global geometry differences between SSM and Transformer representations causally affect robustness, compression, continual memory, or interpretability, or are they mostly reparameterizations with locally similar semantics?

---

## Compact historical / superseded material

These items remain useful context but are not the highest-priority frontier readings:

- **S5 / intermediate S4 variants:** historically useful, but S4 -> LRU -> Mamba -> Mamba-2/3 is a more efficient route to the current SSM line.
- **Intermediate RWKV versions:** useful for engineering history, but RWKV original + RWKV-7 captures the main conceptual trajectory for this research stream.
- **RetNet:** still conceptually valuable for parallel/recurrent/chunkwise equivalence, but currently deprioritized relative to Mamba/GDN/Kimi/RWKV/xLSTM/hybrid lines.
- **Many classical LSTM/GRU variants:** preserve only when a modern paper depends on a specific mechanism; xLSTM is the main active direct continuation of the LSTM lineage.

---

## Bottom line as of 2026-09-12

The token-time frontier did **not** materially change this week: Mamba-3, Gated DeltaNet/KDA/GDN-2/KDN, RWKV-7, xLSTM, and hybrid attention-recurrence remain the major lines, with no new successor that clearly displaces them.

The meaningful movement is in **training and recurrent depth**. `Learning Length-Extrapolatable Recurrent Models` reframes long-horizon recurrent training around **state credit**, while `Thinking with Looped Flows` introduces a new way to train recurrent-depth reasoning using temporally aligned local denoising objectives and probability-flow inference. `Global Divergence, Local Convergence` adds useful evidence that SSMs and Transformers can have substantially different global representation geometry while converging on similar local semantic organization.

The field is therefore broadening from the question "what recurrent update should replace attention?" toward a richer set of questions about **memory type, credit assignment, recurrent computation depth, uncertainty, retrieval, and hybrid specialization**.
