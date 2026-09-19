# RNN and SSM Research Memory

_Last updated: 2026-09-19_

This is a curated living document for modern recurrent sequence models, State Space Models (SSMs), linear attention, recurrent-depth models, and recurrent/attention hybrids in NLP and LLMs. It is organized by the current conceptual structure of the field rather than by scheduler-run order.

## Current assessment

The field is no longer well described as "RNN vs Transformer." The active design space contains at least two importantly different forms of recurrence:

1. **Token-time recurrence**: a fixed-size or bounded state is updated as the model advances through sequence positions. This includes SSMs, linear attention in recurrent form, RWKV, xLSTM, Hawk/Griffin, Gated DeltaNet/KDA, and related hybrids.
2. **Loop-time / recurrent-depth recurrence**: shared computation is repeatedly applied to the same input representation or reasoning state to spend more computation before advancing. This includes Universal/looped Transformers, RecurTrace, Looped Flows, Attractor Models, LSTM-UT, and related latent-reasoning architectures.

These axes should not be conflated. A model can have a long recurrent path across tokens without performing extra computation on the same token, and a looped model can repeatedly refine the same state without carrying a compressed history across arbitrarily many tokens.

For token-time models, the useful decomposition has expanded beyond write/erase/read:

- **write/update**: what new information enters the state;
- **erase/forget**: what old information is attenuated or overwritten;
- **read/query**: how the compressed state is interrogated;
- **uncertainty/confidence**: how strongly the model should trust stored associations;
- **timescale spectrum**: which state directions decay slowly enough to carry long-range information while retaining fast modes for clearing/context switching;
- **serving/reconstruction**: how recurrent states can be cached, reused, quantized, or reconstructed in production.

Hybrid architectures remain especially credible. Recent causal evidence suggests that full attention and recurrent state can learn different memory roles rather than being redundant approximations: attention is strong at exact addressable retrieval/bindings, while recurrence can carry compressed semantic, stylistic, or behavioral context.

A second theme strengthened this week: **more persistent memory is not automatically better**. New recurrent-depth experiments show that an expanding cache can interfere with corrected current states, while bounded gated memory can generalize more robustly. This reinforces the importance of memory selection, gating, and state geometry rather than simply increasing history access.

---

## Material changes since the previous report

### 1. SpectralShift: Effective Context Window Extension of Gated DeltaNet via Spectral Reparameterization

**Status:** NEW — arXiv preprint, first posted 2026-09-13; code released  
**Authors:** Zian Liu, Yiwen Hu, Zican Dong, Tian Xie, Wayne Xin Zhao, Yucheng Ding, Ran Tao, Bryan Dai  
**Primary:** https://arxiv.org/abs/2609.14320  
**Code/results:** https://github.com/RUCAIBox/GDN-SpectralShift

#### Contribution

SpectralShift asks a practical question that becomes central once linear attention is deployed at long context: if a Gated DeltaNet was trained at 8K, why should its learned forgetting dynamics automatically remain appropriate at 64K or 128K?

The paper studies the transition dynamics spectrally and identifies two desirable properties:

1. a sufficiently broad **slow spectral band** whose decay timescale matches the target dependency length;
2. preservation of **fast-decaying modes** needed to clear stale state and switch context.

The method modifies only the GDN alpha projection before long-context continued pretraining. For reference length `L_ref` and target length `L_tar`, the released implementation uses

` s = sqrt(L_ref / L_tar) `

and rescales the centered alpha-projection weights plus the alpha-projection learning rate by `s`. The goal is not to make every gate uniformly slower, but to reshape the distribution of recurrent decay timescales.

#### Relation to prior work

This is not a successor to Gated DeltaNet/KDA/GDN-2 in the sense of a new update rule. It is a **context-extension/training method for the existing recurrent dynamics**. It adds a new axis to the architecture map: state memory is controlled not only by write/erase gates but also by the **spectrum of effective retention timescales**.

It also connects naturally to classical SSM thinking, where eigenvalues/poles determine memory timescales. In that sense, SpectralShift brings an explicitly spectral SSM-style viewpoint back into modern GDN long-context training.

#### Evidence

The released main table evaluates a **1.5B-A0.6B GDN-MoE** base model trained for 500B tokens at 8K and then extended to 32K/64K/128K.

Selected reported results:

- 8K -> 32K, 10B continued-pretraining tokens: RULER at 32K improves **40.6 -> 45.8**.
- staged 64K extension (10B + 10B): RULER at 64K improves **41.6 -> 45.4**.
- staged 128K extension (10B + 10B): mean RULER over 8K/16K/32K/64K/128K improves **52.38 -> 55.18**; general-task average also rises **53.7 -> 54.8**.

The effect is therefore meaningful but not an architecture-level discontinuity.

#### Limitations

- One principal 1.5B GDN-MoE setting; no evidence yet at frontier-scale hybrid models.
- No 1M-context validation.
- The long-context recipe still includes adaptations for the softmax-attention layers, so attribution is not perfectly isolated to GDN.
- It improves retention timescales but does not solve fixed-state capacity or exact binding/retrieval by itself.

#### Assessment

**Incremental architecture-wise, conceptually useful and practically important.** It strengthens the view that long-context recurrent models need explicit control of their decay spectrum. Add **timescale spectrum** to the core memory-design checklist.

---

### 2. Recurrent Looped Transformer (RLT) — original report updated, then independently stress-tested

**Original report:** Yifan Zhang, Jichen Feng, Shihan Qin — technical report, 2026-09-12; updated 2026-09-17  
**Primary:** https://github.com/yifanzhang-pro/recurrent-looped-tranformer  
**Independent evaluation:** Leon Lehmann, Casie Nakamura / Empero AI — 2026-09-15  
**Evaluation page:** https://www.alphaxiv.org/abs/2609.recurrence-looped-transformer-evaluation  
**Code:** https://github.com/empero-org/rlt-evaluation

#### What changed

The initial RLT report was largely an architectural specification. The September 17 update added controlled algorithmic experiments, while an independent evaluation tested the same central idea in ordinary language-model pretraining.

RLT uses a causal encoder plus a decoder that carries its final hidden state and layerwise sliding-window cache from token to token. This creates a recurrent computation path whose depth grows with sequence length.

Important distinction: this is primarily **token-time recurrence**. It does not give one token an arbitrary number of internal loop iterations before advancing; the long path appears because later tokens inherit prior decoder state.

#### Evidence from the updated original report

The updated repository reports small ~25-29M-parameter depth-eight experiments on six algorithmic tasks.

Positive cases include:

- parity: some RLT splits retain **100% accuracy at length 256**, versus about **50%** for the 8-layer Transformer;
- swaps-S5 at 512 operations: one RLT split reports **55.7% final-state accuracy** versus **0.85%** for the Transformer, although variance is large.

But the gains are highly task-dependent:

- 32-digit addition remains poor for all models (~15-17% teacher-forced token accuracy);
- standard S5 remains near floor for all models;
- modular-arithmetic gains are inconsistent and seed-sensitive.

Crucially, the updated report says matched **RLT-0** experiments (same surrounding architecture but hidden-state feedback removed) are still needed to isolate whether the recurrent feedback itself causes the gains. Hardware throughput and RL performance are also not yet measured in that report.

#### Independent controlled language-model evaluation

The Empero evaluation implements exact recurrence and pretrains ~50M and ~140M models on the same 500M-token stream, comparing against parameter-matched Transformers and an RLT ablation with feedback disabled.

Reported results:

- ~50M: RLT and Transformer are effectively tied in held-out loss (**3.8845 vs 3.8841 nats**).
- ~140M: RLT is worse (**3.6884**) than both feedback-disabled RLT (**3.6664**) and the Transformer (**3.6420**).
- the feedback effect does not grow usefully with document position;
- no context-length extrapolation advantage is observed at 1024/2048/4096;
- exact recurrent training remains very expensive even after custom kernels: about **8.4K tok/s** per RTX 5090 versus **124K tok/s** for the Transformer in the reported main setup; the recurrent run used roughly **20x** the GPU-hours.

#### Interpretation

These results do not show that recurrence is useless. They show something narrower and important:

> **A long sequential recurrent path is not, by itself, evidence of useful latent reasoning or efficient effective depth.**

RLT is currently **strengthened for specific algorithmic state-tracking tasks but weakened as a general-purpose language-model architecture claim**. The strongest evidence now argues that its benefits are task- and scale-dependent and that naive per-token recurrence carries a severe training-parallelism cost.

#### Limitations of the negative result

- only ~50M-140M parameter models;
- 500M training tokens;
- one main architectural recipe and dataset mixture;
- does not test billion-scale state-tracking-heavy tasks or more sophisticated recurrent gating/chunking.

#### Assessment

**Important contradictory evidence.** RLT should no longer be treated as a promising architecture on specification alone. Keep it as a test case for the distinction between **temporal path length** and **useful adaptive computation**.

---

### 3. LSTM-UT and Recurrent-Depth Transformers on Cellular Automata

**Status:** NEW — arXiv v1, 2026-09-17  
**Author:** Aras Kavuncu  
**Primary:** https://arxiv.org/abs/2609.19521

#### Contribution

This paper directly studies **memory design across recurrent-depth iterations**. It compares:

- **Block Universal Transformer (BUT):** carries only the current state;
- **CoTFormer:** keeps an expanding cache of prior recurrent-depth states;
- **LSTM Universal Transformer (LSTM-UT):** introduces bounded gated memory.

The key result is counterintuitive: having explicit access to a growing history does not automatically improve recurrent-depth reasoning. On Rule 30 cellular automata, BUT extrapolates to unseen recurrence depths more reliably than CoTFormer. State/cache interventions suggest that CoTFormer's retained history can undermine a corrected current state. In delayed recall, CoTFormer also struggles to select the requested cached representation.

LSTM-UT improves both depth extrapolation and delayed recall over the two baselines in the reported tasks.

#### Relation to RecurTrace / Loop Memory Attention

This is especially relevant to RecurTrace. RecurTrace makes prior loop states explicitly addressable; LSTM-UT shows that **addressability without sufficiently good selection/gating can be harmful**.

That does not contradict RecurTrace directly—the mechanisms and tasks differ—but it weakens a naive interpretation that "keeping more loop states" is intrinsically better. The design question becomes:

- what should be retained;
- how should it be gated/compressed;
- when should old loop states be ignored even if they remain accessible?

#### Limitations

- cellular automata + delayed-recall tasks rather than natural-language pretraining;
- single-author v1 preprint;
- no evidence yet at LLM scale;
- exact numerical gains need to be interpreted within the narrow synthetic setting.

#### Assessment

**Conceptually important, evidence still narrow.** It strengthens the case for **bounded, gated loop-time memory** and adds a useful counterweight to expanding-cache recurrent-depth designs.

---

### 4. The Attention Within: Consensus Dynamics in Selective State Space Models

**Status:** NEW — arXiv v1, 2026-09-16  
**Authors:** João Pedro Silvestre, Álvaro Rodríguez Abella, Paulo Tabuada  
**Primary:** https://arxiv.org/abs/2609.17997

#### Contribution

This is a theoretical/mechanistic SSM paper rather than a new architecture. It asks whether the token-mixing recurrence in selective SSMs has an analogue of the **consensus/over-smoothing dynamics** studied in attention models.

Using a dynamical-systems/ODE view and input-to-state stability arguments, the authors establish local exponential stability of consensus equilibria and characterize a domain of attraction for time-varying weight matrices.

Numerical experiments on pretrained Mamba-2 point to the **output gate** as a mechanism that regulates the approach to consensus and prevents complete collapse.

#### Relation to prior work

- **Mamba-2 / SSD** gives an algebraic/computational bridge between SSMs and attention.
- **Global Divergence, Local Convergence** found different global representation geometry but strong local semantic alignment.
- This paper adds a **dynamical-systems bridge**: the recurrence itself can aggregate tokens toward consensus in a way structurally analogous to attention.

#### Limitations

- local stability theory does not imply every trained SSM will converge to harmful representational collapse;
- numerical validation centers on Mamba-2 and does not establish universality across Mamba-3, GDN, RWKV, etc.;
- no causal demonstration that changing consensus strength improves LM quality.

#### Assessment

**Conceptually important theoretical evidence.** It strengthens the SSM-attention connection beyond implementation-level duality and is especially relevant to studying gates/Jacobians/stability as architecture diagnostics.

---

### 5. Register Tokens for Bounded-State Reasoning in Diffusion Language Models

**Status:** NEW — watchlist/cross-paradigm evidence; arXiv v1, 2026-09-14  
**Authors:** Albert Ge, Chandan Singh, Yufan Zhuang, Xiaodong Liu, Jianfeng Gao, Frederic Sala  
**Primary:** https://arxiv.org/abs/2609.16372

Masked diffusion LMs are not classical RNNs/SSMs, but this paper is relevant to the core bounded-state question. It trains a small number of continuous **register tokens** to carry reasoning progress across generation chunks while previously generated text is cleared.

On LLaDA and Dream, the reported register-state approach beats discrete-text carry on every evaluated benchmark, with gains up to **+8.5 points on math** and **+19.5 on code**.

**Assessment:** keep on the watchlist. It is additional evidence that a learned fixed-size continuous state can preserve useful reasoning progress across chunk boundaries, but it does not yet tell us whether the same mechanism scales to standard autoregressive LLMs or substitutes for token-time recurrent memory.

---

## Active token-time architecture map

### Mamba / SSM line

**S4 -> LRU -> Mamba -> Mamba-2 -> Mamba-3**

- **S4** — structured SSM foundation for efficient long-sequence modeling.  
  Primary: https://arxiv.org/abs/2111.00396
- **LRU** — shows that carefully parameterized/initialized linear recurrence can recover much of modern SSM behavior.  
  Primary: https://arxiv.org/abs/2303.06349
- **Mamba** — selective/input-dependent SSM updates.  
  Primary: https://arxiv.org/abs/2312.00752
- **Mamba-2 / State Space Duality** — stronger algebraic connection between SSMs and structured attention-like matrices plus efficient algorithms.  
  Primary: https://proceedings.mlr.press/v235/dao24a.html
- **Mamba-3** — ICLR 2026 Oral; improved discretization, complex-valued state dynamics/state tracking, MIMO formulation, and inference-first hardware design.  
  Primary: https://arxiv.org/abs/2603.15569

**Current status (2026-09-19): STILL CENTRAL, unchanged architecturally this run.** No new Mamba successor/revision was found that displaces Mamba-3. The new consensus-dynamics paper strengthens the theoretical analysis line around Mamba-2 rather than changing the architecture frontier.

### Linear attention / associative-memory line

**Linear attention -> GLA -> DeltaNet -> Gated DeltaNet -> KDA/Kimi Linear -> Gated DeltaNet-2 / Kalman Delta Networks**

- **Gated DeltaNet** — combines global forgetting with targeted delta-rule associative-memory updates.  
  Primary: https://openreview.net/forum?id=r8H7xhYPwz
- **Kimi Linear / KDA** — channel-wise decay/gating and a large hybrid KDA + MLA architecture.  
  Primary: https://arxiv.org/abs/2510.26692
- **Gated DeltaNet-2** — decouples erase and write decisions.  
  Primary: https://arxiv.org/abs/2605.22791
- **Kalman Delta Networks (KDN)** — adds explicit uncertainty/confidence tracking to associative memory and interprets ordinary delta-style updates as a collapsed Kalman-filter approximation.  
  Primary: https://arxiv.org/abs/2609.07816
- **SpectralShift** — new this run; calibrates GDN retention timescales during long-context extension rather than changing the base update rule.  
  Primary: https://arxiv.org/abs/2609.14320

**Current assessment:** KDN remains the most interesting recent conceptual extension of the update rule; SpectralShift is the most relevant new **long-context training/retention-timescale** result. Neither supersedes the other.

### RWKV

- **RWKV original** — train-parallel / infer-recurrent language modeling.  
  Primary: https://arxiv.org/abs/2305.13048
- **RWKV-7 Goose** — more expressive dynamic-state updates, vector-valued gates, stronger state-tracking/formal-language analysis.  
  Primary: https://arxiv.org/abs/2503.14456

**Current status:** STILL RELEVANT; no material update found this run.

### xLSTM

- **xLSTM** — exponential gating plus scalar/matrix memories, direct modern continuation of the LSTM line.  
  Primary: https://arxiv.org/abs/2405.04517
- **xLSTM 7B** — multi-billion-parameter recurrent LM evidence.  
  Primary: https://arxiv.org/abs/2503.13427
- **xLSTM Scaling Laws** — scaling/compute-quality analysis.  
  Primary: https://openreview.net/forum?id=bpbU549sSg

**Current status:** STILL RELEVANT; no substantive new result found this run.

### Griffin / RecurrentGemma and hybrids

- **Griffin/Hawk** — gated linear recurrence + local attention.  
  Primary: https://arxiv.org/abs/2402.19427
- **RecurrentGemma** — open scaled implementation of the Griffin line.  
  Primary: https://arxiv.org/abs/2404.07839

**Current status:** STILL RELEVANT as early evidence that recurrence and local attention are complementary; no new Griffin/RecurrentGemma update found this run.

---

## Active recurrent-depth / loop-time line

### RecurTrace — still relevant, interpretation refined

**Primary:** https://arxiv.org/abs/2609.03379  
**Status:** arXiv preprint, 2026-09-03

RecurTrace adds **Loop Memory Attention**, allowing each looped layer to query a short explicit history of its own previous recurrent-depth states, plus learned halting.

**Current assessment:** STILL RELEVANT. Looped Flows strengthens the broader recurrent-depth direction, while LSTM-UT adds an important caution: explicitly retaining more loop history can hurt if retrieval/gating is poorly controlled. RecurTrace should therefore be evaluated not only on whether it exposes prior states, but on whether its selection and gating reliably prevent stale-history interference.

### Thinking with Looped Flows — still high priority

**Primary:** https://arxiv.org/abs/2609.11801  
**Status:** arXiv v1, 2026-09-10

Introduces a stateful denoising / probability-flow formulation to improve temporal credit assignment in recurrent-depth reasoning without full long-horizon BPTT. Reported ARC-AGI results substantially exceed TRM under the same architecture in the paper's setting.

**Assessment:** STILL HIGH PRIORITY; no contradictory evidence found this run. Main uncertainty remains transfer to large autoregressive LMs.

### Learning Length-Extrapolatable Recurrent Models / CST — still relevant

**Primary:** https://arxiv.org/abs/2609.09157  
**Status:** arXiv v1, 2026-09-08

Reframes long-horizon recurrent training around **state credit** rather than only parameter-gradient vanishing/explosion and proposes Credit Stabilization through Time.

**Assessment:** STILL CONCEPTUALLY IMPORTANT; awaits replication at larger LM scale.

### Looped GPT-BERT — supporting evidence

**Primary:** https://arxiv.org/abs/2609.09691

Small-model evidence that physical depth can be traded for repeated computation, but with mixed task-level gains and no frontier-scale validation.

### LSTM-UT — new bounded-memory branch

**Primary:** https://arxiv.org/abs/2609.19521

Adds bounded gated memory across loop-time and outperforms current-state-only and expanding-cache baselines on the reported synthetic tasks. Track as a memory-design result rather than a general LLM breakthrough.

### Recurrent Looped Transformer (RLT) — moved to mixed/contested status

**Primary:** https://github.com/yifanzhang-pro/recurrent-looped-tranformer

RLT is included here because it is frequently discussed as "infinite depth," but its recurrence is mostly **across tokens**, not repeated refinement of one token/state before advancing. The latest evidence is mixed: strong synthetic state-tracking in some tasks, but no advantage in an independent 50M-140M language-model study and a severe sequential-training cost.

**Status:** WEAKENED AS A GENERAL LM CLAIM; STILL INTERESTING AS A STATE-TRACKING ARCHITECTURE.

---

## Recent evidence carried forward

### What Attention Recalls and Recurrence Controls in Hybrid Language Models

**Primary:** https://arxiv.org/abs/2609.04434  
**Status:** arXiv preprint, 2026-09-03  
**Assessment:** **STRENGTHENED / still central.**

Causal cache/state interventions in Qwen3.5-4B and Falcon-H1-3B suggest a functional division:

- attention/KV state preserves exact, addressable item retrieval and bindings;
- recurrent state preserves more compressed semantic/behavioral context such as language/persona/style.

This remains one of the strongest empirical arguments for hybrids as **complementary memory systems** rather than simple quality/efficiency compromises.

### Global Divergence, Local Convergence: Representation Geometry in SSMs and Transformers

**Primary:** https://arxiv.org/abs/2609.08692  
**Status:** arXiv v1, 2026-09-08  
**Assessment:** **STILL RELEVANT; now complemented by consensus-dynamics theory.**

Finds strongly different global representation geometry between SSMs and Transformers but much closer local semantic organization. The new consensus-dynamics paper adds a dynamical explanation for how the mechanisms can differ globally while still sharing aggregation behavior.

### Why Gated DeltaNet Survives 4-Bit Quantization

**Primary:** https://arxiv.org/abs/2609.04098  
**Status:** arXiv preprint, 2026-09-03  
**Assessment:** **STILL RELEVANT practical/mechanistic evidence.**

GDN state error does not simply accumulate indefinitely under the tested 4-bit quantization; delta-rule updates can overwrite/correct portions of recurrent-state error.

### Modern Transformers Are Implicit Hybrids

**Primary:** https://arxiv.org/abs/2609.02986  
**Status:** arXiv preprint, 2026-09-02  
**Assessment:** **STILL PROMISING, not yet validated at frontier scale.**

Proposes head-wise assignment of full vs linear attention according to functional role. The hybrid-memory evidence above remains consistent with this direction.

### Curvature-Conditioned Query (CCQ)

**Primary:** https://arxiv.org/abs/2606.01294  
**Status:** EMNLP 2026 camera-ready revision reported 2026-08-30  
**Assessment:** **STILL RELEVANT.**

Shifts focus from write/erase dynamics to **read dynamics**: how a query interrogates compressed linear-attention state.

### Tail-Replay

**Primary:** https://arxiv.org/abs/2608.30310  
**Status:** arXiv preprint, 2026-08-31  
**Assessment:** **STILL RELEVANT as serving/system evidence.**

Treats gated linear-attention state as structured lossy compression and reconstructs matched-prefix state by replaying only a recent suffix.

---

## Peripheral / watchlist

### Register Tokens for Bounded-State Reasoning in Diffusion Language Models

**Primary:** https://arxiv.org/abs/2609.16372  
**Status:** 2026-09-14 arXiv v1  
**Assessment:** WATCHLIST — useful cross-paradigm bounded-state evidence, not a direct RNN/SSM result.

### RiLM: Parameter-Efficient Language Modeling via Geodesic Decoding

**Primary:** https://arxiv.org/abs/2609.10305  
**Status:** 2026-09-09 arXiv v1

Uses a recurrent trajectory on a Riemannian manifold and geodesic vocabulary decoding. Interesting geometry, but current evidence is tiny/sub-million-parameter.

**Assessment:** DEPRIORITIZED / WATCHLIST.

### ConvMem: Convolutional Memory for Long-Context Reasoning

**Primary:** https://arxiv.org/abs/2609.10441  
**Status:** 2026-09-09 arXiv v1

Hierarchical/tree-style context summarization rather than a new recurrent sequence-model architecture.

**Assessment:** PERIPHERAL to this stream.

---

## Main open questions

1. **State capacity vs exact recall:** how much exact binding/retrieval can fixed-state models recover without reintroducing explicit addressable memory?
2. **Functional specialization in hybrids:** can routing between full attention and recurrent/linear memory be learned rather than hard-coded by layer/head?
3. **Uncertainty-aware memory:** does KDN-style confidence/covariance tracking remain useful at 7B+ and frontier-scale training?
4. **Read dynamics:** can CCQ-like query conditioning close a meaningful fraction of the recall gap without increasing state size?
5. **Retention spectrum:** can SpectralShift-style timescale calibration generalize beyond GDN to KDA, Mamba, RWKV, xLSTM, or learned hybrid routing?
6. **Length extrapolation:** does state-credit stabilization remain effective for large LMs on real long-context corpora?
7. **Recurrent-depth memory:** when should loop history be explicit, compressed, gated, or discarded? LSTM-UT now makes stale-history interference a first-class concern.
8. **Adaptive compute vs mere path length:** what measurements distinguish genuinely useful recurrent reasoning from simply creating a longer sequential graph? RLT's mixed evidence makes this urgent.
9. **Token-time vs loop-time composition:** can a recurrent/SSM token mixer itself use adaptive recurrent depth without destroying training/inference efficiency?
10. **Hardware reality:** which theoretically linear/recurrent mechanisms still win after kernels, batching, prefix caching, quantization, and sequential dependencies are included?
11. **Geometry and function:** do SSM/Transformer global-geometry differences and consensus dynamics causally affect robustness, memory, interpretability, or continual learning?
12. **Scaling validity:** many new positive results still sit in the 25M-1.5B range. Which mechanisms survive 7B-70B and trillion-token training?

---

## Compact historical / superseded material

These remain useful context but are not the highest-priority frontier readings:

- **S5 / intermediate S4 variants:** historically useful, but S4 -> LRU -> Mamba -> Mamba-2/3 is the more efficient current study route.
- **Intermediate RWKV versions:** useful engineering history; RWKV original + RWKV-7 captures most of the conceptual trajectory for this stream.
- **RetNet:** conceptually valuable for parallel/recurrent/chunkwise equivalence, but currently deprioritized relative to Mamba/GDN/Kimi/RWKV/xLSTM/hybrid lines.
- **Many classical LSTM/GRU variants:** retain only when a modern paper depends on a specific mechanism; xLSTM is the main active direct continuation of the LSTM lineage.
- **Naive "more recurrent cache is better" framing:** weakened by LSTM-UT's state/cache interventions; memory selection/gating must be treated as part of the architecture, not an afterthought.
- **RLT as evidence-free architecture speculation:** superseded. The original report now has synthetic experiments and an independent LM evaluation exists; the correct status is mixed/contested rather than untested.

---

## Bottom line as of 2026-09-19

The core frontier models themselves did **not** receive a clear successor this week: Mamba-3, RWKV-7, xLSTM, Gated DeltaNet/KDA/GDN-2/KDN, Griffin/RecurrentGemma, and hybrid attention-recurrence remain the main established lines.

The meaningful changes are more structural:

1. **SpectralShift** adds explicit **retention-timescale/spectral calibration** to long-context GDN training and produces consistent 32K-128K retrieval gains.
2. **RLT now has contradictory evidence:** the updated original report shows strong gains on some algorithmic state-tracking tasks, while an independent ordinary-LM evaluation finds no quality benefit at 50M-140M and a large sequential-training penalty. This weakens the idea that longer recurrent paths automatically produce useful reasoning depth.
3. **LSTM-UT** shows that expanding loop-history caches can interfere with corrected state, while bounded gated memory can work better on recurrent-depth extrapolation/recall.
4. **The Attention Within** strengthens the theoretical bridge between selective SSM recurrence and Transformer attention via consensus dynamics, with the output gate appearing to regulate representational collapse in Mamba-2.

The field is therefore moving away from a single question—"which linear/recurrent mixer replaces attention?"—toward a richer architecture science of **memory type, decay spectrum, confidence, read/write rules, bounded vs addressable loop memory, credit assignment, adaptive computation, and real hardware cost**.