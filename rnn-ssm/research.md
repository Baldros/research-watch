# RNN and SSM Research Memory

_Last updated: 2026-09-19_

This is a curated living document for modern recurrent sequence models, State Space Models (SSMs), linear attention, recurrent-depth models, and recurrent/attention hybrids in NLP and LLMs. It is organized by the current structure of the field rather than by scheduler-run order.

## Executive assessment

The field is no longer well described as **RNN vs Transformer**. Two different recurrence axes now matter:

1. **Token-time recurrence** — a bounded state is updated as the model advances through sequence positions. This includes SSMs, linear attention in recurrent form, RWKV, xLSTM, Gated DeltaNet/KDA, Hawk/Griffin, and related hybrids.
2. **Loop-time / recurrent-depth recurrence** — shared computation is applied repeatedly to the same representation before advancing, allowing parameter reuse and test-time compute scaling. This includes looped Transformers, RecurTrace, Looped Flows, LSTM-UT, and related latent-reasoning architectures.

These axes should not be conflated. A long recurrent path across tokens does not automatically mean that the model performs useful extra computation on a fixed token or latent state.

For token-time memory, the useful decomposition has expanded to:

- **write/update** — what enters the state;
- **erase/forget** — what old information is attenuated or overwritten;
- **read/query** — how compressed state is interrogated;
- **uncertainty/confidence** — how strongly the model trusts stored associations;
- **timescale spectrum** — which state directions decay slowly or quickly;
- **serving/reconstruction** — how recurrent state is cached, quantized, reused, or reconstructed.

For loop-time memory, the emerging design questions are:

- whether prior loop states should remain explicitly addressable;
- how stale intermediate states are gated or discarded;
- how temporal credit is assigned across loops;
- how many loops should execute;
- how repeated depth can be accelerated at inference.

**Core frontier status:** no new successor this run displaced Mamba-3, RWKV-7, xLSTM, Gated DeltaNet/KDA/GDN-2/KDN, Kimi Linear, Griffin, or RecurrentGemma. The meaningful changes were in long-context adaptation, looped-model inference, controlled evaluation, and memory dynamics.


## Conceptual taxonomy used by this research stream

The earlier working abstraction of **“modern RNNs”** has now been refined into distinct lineages. The umbrella term used here is **modern recurrent models**, with five main architectural families:

1. **RNN-native / gated recurrent architectures** — classical RNN/LSTM descendants such as xLSTM, plus recurrent-native designs such as RWKV.
2. **State Space Models (SSMs)** — S4/LRU/Mamba-style models, where long memory is framed primarily through state dynamics, stability, selectivity, and timescale spectrum.
3. **Recurrent linear attention / associative memory** — DeltaNet, Gated DeltaNet, KDA, GDN-2, KDN, and related work, where the state is treated explicitly as finite associative memory and research focuses on write, erase, read, uncertainty, and interference.
4. **Recurrent-attention hybrids** — Griffin/RecurrentGemma, Kimi Linear and related architectures that divide labor between compressed recurrent state and explicit addressable attention/KV memory.
5. **Recurrent depth / loop-time models** — RecurTrace, Looped Flows, LSTM-UT and related models, where recurrence happens across repeated internal computation rather than merely across token positions.

This refinement is important because **“improving long-term memory” is not a single research program**. Different papers target different problems: retention, capacity, write/erase rules, read dynamics, uncertainty, credit assignment, explicit addressability, or serving/reconstruction.

A detailed conceptual map, including how the long-memory papers belong to each lineage, is maintained separately in [`taxonomy.md`](./taxonomy.md).

---

# Material changes since the previous run

## 1. SpectralShift: Effective Context Window Extension of Gated DeltaNet via Spectral Reparameterization

**Authors:** Zian Liu, Yiwen Hu, Zican Dong, Tian Xie, Wayne Xin Zhao, Yucheng Ding, Ran Tao, Bryan Dai  
**Date/status:** arXiv preprint, submitted 2026-09-13; updated 2026-09-15; code released  
**Primary:** https://arxiv.org/abs/2609.14320  
**Code/results:** https://github.com/RUCAIBox/GDN-SpectralShift

### Technical contribution

SpectralShift studies why a Gated DeltaNet trained at 8K context may not automatically have appropriate forgetting dynamics at 64K or 128K. It analyzes GDN transition dynamics spectrally and argues that long-context retrieval requires both:

1. a broad **slow spectral band** aligned with the target dependency length;
2. preservation of **fast-decaying modes** for state clearing and context switching.

The method reparameterizes the GDN alpha projection before continued pretraining and scales its learning rate. In the released implementation,

```text
s = sqrt(L_ref / L_tar)
W_alpha <- mean(W_alpha) + s * (W_alpha - mean(W_alpha))
eta_alpha <- s * eta_base
```

The goal is not to make every gate uniformly slower. It reshapes the distribution of effective retention timescales.

### Relation to prior work

This is not a new GDN update rule and does not supersede Gated DeltaNet, KDA, GDN-2, or KDN. It is a **context-extension method** for an existing recurrent mixer.

Conceptually, it restores a classical SSM perspective: memory length is controlled not only by gates and write rules, but by the spectrum of recurrent transition timescales.

### Evidence

The principal experiment uses a **1.5B-A0.6B GDN-MoE** base model trained for 500B tokens at 8K and then extended to 32K/64K/128K.

Selected reported results:

- 8K -> 32K with 10B continued-pretraining tokens: RULER at 32K improves **40.6 -> 45.8**.
- staged 64K extension (10B + 10B): RULER at 64K improves **41.6 -> 45.4**.
- staged 128K extension (10B + 10B): mean RULER across 8K/16K/32K/64K/128K improves **52.38 -> 55.18**; general-task average rises **53.7 -> 54.8**.

### Limitations

- One main 1.5B GDN-MoE setting.
- No frontier-scale or 1M-context validation.
- The extension recipe also adapts the model's softmax-attention components, so attribution is not perfectly isolated to GDN.
- It improves retention timescales, not fixed-state capacity or exact associative binding by itself.

### Assessment

**Incremental architecturally, but conceptually useful and practically important.** It adds **timescale-spectrum calibration** to the main design checklist for recurrent long-context models.

---

## 2. LoopSpec: Pipelined Self-Speculative Decoding for Looped Transformers

**Authors:** SangLyul Cho, Langqing Cui, Sehoon Kim, Dongsu Han, Insu Han  
**Date/status:** arXiv v1, 2026-09-15  
**Primary:** https://arxiv.org/abs/2609.17184  
**Code:** https://github.com/kaist-flexml-lab/loopspec

### Technical contribution

Looped Transformers repeatedly apply the same block across recurrent depths. This saves parameter memory but increases decoding latency because all recurrent depths normally finish for token `t` before token `t+1` starts.

LoopSpec treats intermediate recurrent-depth predictions as **self-generated speculative drafts**:

1. an early recurrent state proposes the next token;
2. computation for the speculative child prefix starts immediately;
3. the original branch continues to final depth and verifies the draft;
4. accepted work is retained; rejected branches are pruned;
5. an optional deeper second proposal creates a fallback branch when the first proposal is likely to fail.

The method is training-free and preserves the final model distribution under greedy and sampling decoding through rejection-sampling correction.

### Why it matters

LoopSpec converts a property usually treated as overhead—many progressively refined intermediate loop states—into a built-in draft model. It therefore addresses one of the strongest practical objections to recurrent depth: repeated parameter reuse can reduce model size while making generation memory-bandwidth bound and slow.

It also provides a useful mechanistic observation: intermediate recurrent-depth distributions approach the final distribution progressively enough to serve as drafts. The paper reports, for one Raven setting, depth-1 top-1 agreement above 86% on GSM8K and above 99% by depth 8.

### Evidence

The paper evaluates seven checkpoints from two looped-model families and reports:

- up to **6.83x lossless inference speedup**;
- up to **1.7x** over the training-based DFlash comparison;
- closed-form proposal-depth selection based on an empirical power-law decay of intermediate-to-final distribution distance.

### Limitations

- Does not improve model quality; it only accelerates compatible looped models.
- Speedup depends on draft acceptance, model family, recurrent depth, implementation, batching, and hardware.
- The strongest number is an upper result across tested checkpoints, not a universal expected gain.
- It does not solve recurrent-depth training stability or memory selection.

### Assessment

**Incremental in architecture, important in systems.** LoopSpec materially strengthens the deployment case for looped Transformers and should remain in the active recurrent-depth systems line.

---

## 3. Recurrent Looped Transformer (RLT): original report updated and independently stress-tested

**Original report:** Yifan Zhang, Jichen Feng, Shihan Qin  
**Status:** technical report dated 2026-09-12; repository updated 2026-09-17  
**Primary:** https://github.com/yifanzhang-pro/recurrent-looped-tranformer

**Independent evaluation:** Leon Lehmann, Casie Nakamura / Empero AI  
**Date/status:** public controlled evaluation, 2026-09-15; not peer reviewed  
**Evaluation:** https://www.alphaxiv.org/abs/2609.recurrence-looped-transformer-evaluation  
**Code:** https://github.com/empero-org/rlt-evaluation

### Architecture and classification

RLT uses a causal encoder plus a decoder that carries its final hidden state and layerwise sliding-window cache from token to token. The recurrent path grows with sequence length.

Despite the name, this is primarily **token-time recurrence**, not arbitrary recurrent-depth refinement of one token. Later tokens inherit more prior recurrent computation, but a fixed number of decoder blocks runs per token.

### Updated evidence from the original report

The September 17 update adds small ~25-29M-parameter algorithmic experiments.

Positive cases include:

- parity: some RLT splits retain **100% accuracy at length 256**, versus about **50%** for an 8-layer Transformer;
- swaps-S5 at 512 operations: one RLT split reports **55.7% final-state accuracy** versus **0.85%** for the Transformer, with high variance.

But the gains are task dependent:

- 32-digit addition remains poor for every model;
- standard S5 remains near floor;
- modular-arithmetic results are inconsistent and seed sensitive.

The report itself notes that matched RLT-0 experiments are still needed to isolate hidden-state feedback from the broader encoder-decoder design.

### Independent language-model evaluation

The independent study trains ~50M and ~140M models on the same 500M-token stream and compares RLT with parameter-matched Transformers and feedback-disabled RLT.

Reported results:

- ~50M: RLT and Transformer are effectively tied in held-out loss (**3.8845 vs 3.8841 nats**).
- ~140M: RLT is worse (**3.6884**) than feedback-disabled RLT (**3.6664**) and the Transformer (**3.6420**).
- no useful context-length extrapolation advantage at 1024/2048/4096.
- main recurrent throughput: about **8.4K tok/s per RTX 5090**, versus **124K** for the Transformer.
- roughly **22 GPU-hours** to process the training stream for RLT, versus **1.14 GPU-hours** for the Transformer in the reported setup.

### Interpretation

The evidence now supports a narrower conclusion:

> A longer sequential recurrent path is not, by itself, evidence of useful latent reasoning or efficient effective depth.

RLT is **strengthened for some algorithmic state-tracking tasks** but **weakened as a general-purpose language-model claim**.

### Limitations of the negative evidence

- only ~50M-140M models;
- 500M training tokens;
- one principal data mixture and implementation family;
- does not rule out better feedback gates, chunked recurrence, or billion-scale state-tracking-specialized models.

### Assessment

**Mixed/contested.** Retain RLT as an important controlled test case for distinguishing temporal path length from genuinely useful adaptive computation.

---

## 4. LSTM-UT and Recurrent-Depth Transformers on Cellular Automata

**Author:** Aras Kavuncu  
**Date/status:** arXiv v1, 2026-09-17  
**Primary:** https://arxiv.org/abs/2609.19521

### Technical contribution

The paper compares three loop-time memory strategies:

- **Block Universal Transformer:** only the current hidden state persists;
- **CoTFormer:** an expanding cache exposes previous recurrent-depth states;
- **LSTM Universal Transformer:** bounded gated memory persists across recurrent-depth iterations.

On Rule 30 cellular automata, the current-state-only baseline extrapolates to unseen recurrent depths more reliably than the expanding-cache model. State/cache interventions suggest that retained old states can interfere with a corrected current state. In delayed recall, CoTFormer also fails to reliably select the requested cached representation.

LSTM-UT improves both depth extrapolation and delayed recall in the reported tasks.

### Relation to RecurTrace / Loop Memory Attention

This is a useful caution for RecurTrace-like explicit loop-history access:

> Making previous loop states addressable is not sufficient; selection and gating determine whether that history helps or contaminates the current computation.

The results do not directly contradict RecurTrace because the mechanisms and tasks differ, but they weaken the naive belief that more loop history is intrinsically better.

### Limitations

- cellular automata and delayed-recall tasks, not natural-language pretraining;
- single-author v1 preprint;
- no billion-scale evidence;
- the result establishes an inductive-bias point, not a general LLM victory.

### Assessment

**Conceptually important; empirically narrow.** It strengthens the case for bounded, gated loop-time memory.

---

## 5. The Attention Within: Consensus Dynamics in Selective State Space Models

**Authors:** João Pedro Silvestre, Álvaro Rodríguez Abella, Paulo Tabuada  
**Date/status:** arXiv v1, 2026-09-16  
**Primary:** https://arxiv.org/abs/2609.17997

### Technical contribution

This is a theoretical/mechanistic SSM study. It asks whether selective-SSM recurrence drives token representations toward **consensus**, analogous to attention-induced clustering/over-smoothing in Transformers.

Using an ODE/dynamical-systems view and input-to-state stability arguments, the authors establish local exponential stability of consensus equilibria and characterize a domain of attraction for time-varying weight matrices.

Numerical experiments on a pretrained Mamba-2 model point to the **output gate** as a regulator that prevents complete consensus collapse.

### Relation to prior work

- Mamba-2 / State Space Duality links SSM and attention algebraically/computationally.
- `Global Divergence, Local Convergence` compares their learned representation geometry.
- This paper adds a **dynamical bridge**: selective recurrence can aggregate tokens toward consensus in a structurally similar way to attention.

### Limitations

- Local stability does not imply every trained SSM reaches harmful collapse.
- Numerical evidence centers on Mamba-2.
- No causal result yet shows that changing consensus strength improves LM quality.

### Assessment

**Conceptually important theoretical evidence.** Especially relevant to Jacobian, stability, gate, and representation-geometry analysis.

---

## 6. Long-Context Demonstration Selection Using State Space Models

**Authors:** Ziniu Zhang, Zhenshuo Zhang, Ruoxuan Xiong, Gene Cooperman, Hongyang R. Zhang  
**Date/status:** arXiv preprint, submitted 2026-09-15; updated 2026-09-17  
**Primary:** https://arxiv.org/abs/2609.17888

The paper distills consecutive Transformer layer groups into small SSMs and uses the resulting representations for selecting in-context demonstrations. It reports less than **0.7% approximation error** relative to the target outputs, **14.2x FLOP reduction**, and **+6.48% accuracy** over baseline demonstration-selection methods on the reported tasks.

**Assessment:** useful application evidence, but peripheral to the core architecture frontier. It demonstrates that SSMs can serve as efficient learned surrogates for long-context selection; it does not establish a new language-model backbone.

---

# Active architecture map

## A. Mamba / selective-SSM line

**S4 -> LRU -> Mamba -> Mamba-2 -> Mamba-3**

- **S4:** https://arxiv.org/abs/2111.00396
- **LRU:** https://arxiv.org/abs/2303.06349
- **Mamba:** https://arxiv.org/abs/2312.00752
- **Mamba-2 / State Space Duality:** https://proceedings.mlr.press/v235/dao24a.html
- **Mamba-3:** https://arxiv.org/abs/2603.15569

**Status:** **STILL CENTRAL; no successor this run.** Mamba-3 remains the current endpoint. The new consensus-dynamics work strengthens the theoretical analysis around Mamba-2 but does not move the architecture frontier.

## B. Linear attention / associative memory

**Linear attention -> GLA -> DeltaNet -> Gated DeltaNet -> KDA/Kimi Linear -> GDN-2 / KDN**

- **Gated DeltaNet:** https://openreview.net/forum?id=r8H7xhYPwz
- **Kimi Linear / KDA:** https://arxiv.org/abs/2510.26692
- **Gated DeltaNet-2:** https://arxiv.org/abs/2605.22791
- **Kalman Delta Networks:** https://arxiv.org/abs/2609.07816
- **SpectralShift:** https://arxiv.org/abs/2609.14320

**Current assessment:**

- KDN remains the most interesting recent **update-rule** extension because it adds confidence/uncertainty tracking.
- GDN-2 remains important for decoupling erase and write.
- SpectralShift is the main new **long-context timescale-calibration** result.
- These are complementary rather than mutually superseding.

## C. RWKV

- **RWKV original:** https://arxiv.org/abs/2305.13048
- **RWKV-7 Goose:** https://arxiv.org/abs/2503.14456

**Status:** **STILL RELEVANT; no material update this run.**

## D. xLSTM

- **xLSTM:** https://arxiv.org/abs/2405.04517
- **xLSTM 7B:** https://arxiv.org/abs/2503.13427
- **xLSTM Scaling Laws:** https://openreview.net/forum?id=bpbU549sSg

**Status:** **STILL RELEVANT; no substantive new result this run.**

## E. Griffin / RecurrentGemma and recurrent-attention hybrids

- **Griffin/Hawk:** https://arxiv.org/abs/2402.19427
- **RecurrentGemma:** https://arxiv.org/abs/2404.07839

**Status:** **STILL RELEVANT.** They remain early strong evidence that local/full attention and recurrence can be complementary. No new Griffin/RecurrentGemma update was found.

---

# Active recurrent-depth / loop-time line

## RecurTrace — still relevant, interpretation refined

**Primary:** https://arxiv.org/abs/2609.03379

RecurTrace adds Loop Memory Attention, making a short history of previous loop states explicitly addressable, plus adaptive halting.

**Status:** **STILL RELEVANT.** LSTM-UT adds an important constraint: explicit loop history must be selected and gated well enough to prevent stale-state interference.

## Thinking with Looped Flows — still high priority

**Primary:** https://arxiv.org/abs/2609.11801

Uses stateful denoising and probability-flow training to improve temporal credit assignment in recurrent-depth reasoning without full long-horizon BPTT.

Reported under the same TRM architecture:

- ARC-AGI-1: **58.8% vs 44.6%** for TRM;
- ARC-AGI-2: **12.2% vs 7.8%** for TRM.

**Status:** **STILL HIGH PRIORITY.** Main uncertainty is transfer to general autoregressive LM pretraining.

## Learning Length-Extrapolatable Recurrent Models / CST

**Primary:** https://arxiv.org/abs/2609.09157

Reframes long-horizon recurrent training around **state credit** rather than only parameter-gradient vanishing/explosion and proposes Credit Stabilization through Time. Reports gains up to evaluation lengths 128x the training length.

**Status:** **STILL CONCEPTUALLY IMPORTANT; awaits large-LM replication.**

## LoopSpec — new systems branch

**Primary:** https://arxiv.org/abs/2609.17184

Uses intermediate recurrent-depth predictions as lossless speculative drafts and pipelines work across tokens.

**Status:** **NEW, practically important.**

## LSTM-UT — new bounded-memory branch

**Primary:** https://arxiv.org/abs/2609.19521

Shows, in synthetic recurrent-depth tasks, that bounded gated memory can outperform both current-state-only and expanding-cache approaches.

**Status:** **NEW, conceptually useful but narrow evidence.**

## Looped GPT-BERT

**Primary:** https://arxiv.org/abs/2609.09691

Small-model evidence that physical depth can be traded for repeated computation, but with mixed task-level gains.

**Status:** **SUPPORTING EVIDENCE, not frontier-scale validation.**

## Recurrent Looped Transformer

**Primary:** https://github.com/yifanzhang-pro/recurrent-looped-tranformer

**Status:** **WEAKENED AS A GENERAL LM CLAIM; STILL INTERESTING FOR ALGORITHMIC STATE TRACKING.**

---

# Important evidence carried forward

## What Attention Recalls and Recurrence Controls in Hybrid Language Models

**Primary:** https://arxiv.org/abs/2609.04434  
**Status:** **STRENGTHENED / still central.**

Causal cache/state interventions suggest:

- attention/KV state preserves exact, addressable retrieval and bindings;
- recurrent state preserves compressed semantic/behavioral context such as language, persona, and style.

This remains one of the strongest empirical arguments for hybrids as complementary memory systems.

## Global Divergence, Local Convergence

**Primary:** https://arxiv.org/abs/2609.08692  
**Status:** **STILL RELEVANT; now complemented by consensus-dynamics theory.**

Reports substantially different global representation geometry between SSMs and Transformers but similar local semantic organization.

## Why Gated DeltaNet Survives 4-Bit Quantization

**Primary:** https://arxiv.org/abs/2609.04098  
**Status:** **STILL RELEVANT practical/mechanistic evidence.**

Shows that, in the tested 27B hybrid, GDN-state error does not simply accumulate under full 4-bit quantization; delta-rule updates can overwrite/correct recurrent-state error.

## Modern Transformers Are Implicit Hybrids

**Primary:** https://arxiv.org/abs/2609.02986  
**Status:** **STILL PROMISING; not frontier-scale validated.**

Proposes head-wise rather than layer-wise assignment of full versus linear attention according to functional role.

## Curvature-Conditioned Query

**Primary:** https://arxiv.org/abs/2606.01294  
**Status:** EMNLP 2026 camera-ready; **STILL RELEVANT.**

Moves attention from write/erase dynamics to **read dynamics**: how the query interrogates compressed memory.

## Tail-Replay

**Primary:** https://arxiv.org/abs/2608.30310  
**Status:** **STILL RELEVANT as serving evidence.**

Treats gated linear-attention state as structured lossy compression and reconstructs shared-prefix state by replaying only a recent suffix.

---

# Peripheral / watchlist

## Register Tokens for Bounded-State Reasoning in Diffusion Language Models

**Primary:** https://arxiv.org/abs/2609.16372

Carries a small continuous register state across diffusion-LM generation chunks while clearing generated text. Reports improvements up to +8.5 points on math and +19.5 on code over discrete-text carry.

**Assessment:** useful cross-paradigm evidence for bounded continuous reasoning state, but not a direct RNN/SSM architecture result.

## RiLM: Parameter-Efficient Language Modeling via Geodesic Decoding

**Primary:** https://arxiv.org/abs/2609.10305

Interesting recurrent/Riemannian geometry idea, but evidence remains sub-million-parameter and controlled.

**Status:** **DEPRIORITIZED / WATCHLIST.**

## ConvMem

**Primary:** https://arxiv.org/abs/2609.10441

Hierarchical/tree-style context summarization rather than a new recurrent sequence-model backbone.

**Status:** **PERIPHERAL.**

---

# Open questions

1. **State capacity vs exact recall:** how much exact binding can fixed-state models recover without explicit addressable memory?
2. **Hybrid specialization:** can routing between full attention and recurrent memory be learned instead of fixed by layer/head?
3. **Uncertainty-aware memory:** does KDN remain useful at 7B-70B scale?
4. **Read dynamics:** can CCQ-style query conditioning materially close the recall gap?
5. **Retention spectrum:** does SpectralShift generalize beyond GDN to KDA, Mamba, RWKV, or xLSTM?
6. **Loop-history selection:** when should prior loop states be explicit, compressed, gated, or discarded?
7. **Adaptive compute vs path length:** what proves that recurrence performs useful extra reasoning rather than merely extending the sequential graph?
8. **Looped-model inference:** do LoopSpec gains remain large under production batching, different acceptance rates, and other looped families?
9. **Token-time + loop-time composition:** can an SSM/linear-attention mixer use adaptive recurrent depth without losing its efficiency advantage?
10. **Hardware reality:** which theoretically linear/recurrent mechanisms still win after kernels, batching, prefix caching, quantization, and sequential dependencies?
11. **Geometry and stability:** do consensus dynamics and global geometry differences causally affect robustness, memory, or interpretability?
12. **Scaling validity:** many positive results remain in the 25M-1.5B range. Which survive trillion-token, 7B-70B training?

---

# Compact historical / deprioritized material

- **S5 and intermediate S4 variants:** useful historically, but S4 -> LRU -> Mamba -> Mamba-2/3 is the more efficient current study route.
- **Intermediate RWKV releases:** useful engineering history; RWKV original + RWKV-7 captures the main conceptual arc.
- **RetNet:** still valuable for parallel/recurrent/chunkwise equivalence, but deprioritized relative to Mamba, GDN/KDA, RWKV, xLSTM, and modern hybrids.
- **Many classical LSTM/GRU variants:** retain only when a modern result depends on a specific mechanism; xLSTM is the main active direct continuation.
- **Naive “more persistent memory is better”:** weakened by LSTM-UT's cache/state interventions.
- **RLT as unevaluated speculation:** superseded. It now has original synthetic experiments and an independent LM evaluation; the correct status is mixed/contested.

---

# Bottom line as of 2026-09-19

The main architecture endpoints remain stable: Mamba-3, RWKV-7, xLSTM, Gated DeltaNet/KDA/GDN-2/KDN, Kimi Linear, Griffin/RecurrentGemma, and hybrid attention-recurrence remain the central lines.

The meaningful changes this run are:

1. **SpectralShift** adds explicit retention-timescale calibration for extending GDN context windows.
2. **LoopSpec** makes recurrent-depth intermediate states operationally useful as lossless speculative drafts, substantially improving the deployment story for looped Transformers.
3. **RLT receives mixed evidence:** strong gains on some algorithmic state-tracking tasks, but no ordinary-LM benefit at 50M-140M in an independent study and a severe sequential-training penalty.
4. **LSTM-UT** shows that explicit expanding loop history can interfere with corrected state; bounded gated memory may be a better inductive bias.
5. **The Attention Within** strengthens the theoretical SSM-attention connection through consensus dynamics and gate-regulated stability.

The field is moving from the single question “which recurrent mixer replaces attention?” toward an architecture science of **memory type, decay spectrum, confidence, read/write rules, bounded versus addressable loop memory, credit assignment, adaptive computation, and real hardware cost**.