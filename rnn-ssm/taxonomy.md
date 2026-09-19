# Conceptual Taxonomy of Modern Recurrence for NLP and LLMs

_Last updated: 2026-09-19_

This document captures the conceptual framework used by the `rnn-ssm` research stream. It complements `research.md`: the research file tracks the current evidence and frontier papers, while this file records the more stable taxonomy and the way we interpret memory mechanisms across architectural lineages.

## 1. Why this taxonomy exists

The research stream initially used **RNN** as a broad conceptual umbrella for the return of recurrence in NLP: models that carry state instead of relying only on a growing Transformer KV cache.

That abstraction was pedagogically useful, but it becomes too coarse once we compare mechanisms. Modern recurrent NLP now contains several distinct mathematical lineages that solve the memory problem differently.

The refined view is:

```text
modern recurrence for NLP/LLMs
|
|-- recurrent neural architectures
|   |-- LSTM / GRU lineage
|   |-- xLSTM
|   `-- RWKV and related recurrent-native designs
|
|-- state space models (SSMs)
|   |-- S4 / S5 / LRU bridge
|   `-- Mamba -> Mamba-2 -> Mamba-3
|
|-- recurrent linear attention / associative memory
|   |-- GLA
|   |-- DeltaNet
|   |-- Gated DeltaNet
|   |-- KDA / Kimi Linear
|   |-- Gated DeltaNet-2
|   `-- Kalman Delta Networks
|
|-- recurrent-attention hybrids
|   |-- Griffin / RecurrentGemma
|   |-- Jamba / Samba / Falcon-H1
|   `-- Kimi Linear and related mixed-memory designs
|
`-- recurrent depth / loop-time recurrence
    |-- Universal / looped Transformers
    |-- RecurTrace
    |-- Looped Flows
    `-- LSTM-UT
```

These categories overlap at the boundaries. They are best understood as **research lineages and dominant mathematical viewpoints**, not mutually exclusive boxes.

---

## 2. First split: token-time versus loop-time recurrence

Before distinguishing families, there is a more fundamental axis.

### Token-time recurrence

The state advances with sequence position:

```math
h_t = F(h_{t-1}, x_t).
```

The main question is:

> What information from previous tokens should survive in the current state?

This includes RNNs, SSMs, recurrent linear attention, RWKV, and the recurrent component of hybrid models.

### Loop-time / recurrent-depth recurrence

The input/token position may remain fixed while the model repeatedly refines an internal state:

```math
z^{(r)} = G(z^{(r-1)}, x).
```

The main question is:

> What information from previous internal computation steps should survive or remain addressable?

This includes looped Transformers, RecurTrace, Looped Flows, LSTM-UT, and related latent-reasoning systems.

A model can combine both axes. They should not be conflated.

---

## 3. The long-term-memory problem is not one problem

When a paper says it improves "memory", identify which subproblem it actually attacks.

### 3.1 Retention

Can information survive many recurrent steps?

```text
old information -> state dynamics -> still recoverable later?
```

### 3.2 Capacity

How much information can a fixed-size state contain before interference becomes severe?

This is the fundamental pressure behind the recall-throughput tradeoff.

### 3.3 Write

What new information should enter the state, and how strongly?

### 3.4 Erase / overwrite

What should be removed or attenuated before new information is written?

### 3.5 Read

Given a compressed state, how should a query recover the relevant information?

### 3.6 Uncertainty / confidence

How strongly should the model trust an existing memory before overwriting it?

### 3.7 Credit assignment / trainability

Can losses far in the future train states and transitions far in the past?

### 3.8 Explicit addressability

Should old information remain individually accessible, or only through a compressed summary?

### 3.9 Serving and reconstruction

Can recurrent states be cached, reconstructed, quantized, or reused efficiently in production?

These subproblems explain why papers that all claim "better long-term memory" can belong to very different lineages.

---

# 4. Lineage A — recurrent neural architectures

## Core idea

Classical RNNs directly learn a nonlinear recurrent update:

```math
h_t = \sigma(W_h h_{t-1} + W_x x_t).
```

The historical memory problem is that useful information and gradients can decay or become unstable over long horizons.

## Main memory strategy

**Gating and dedicated recurrent memory structure.**

LSTM introduced explicit gates for write/forget/read-like control. Modern descendants refine the same basic idea.

### xLSTM

Primary: https://arxiv.org/abs/2405.04517

xLSTM modernizes the LSTM line with exponential gating and scalar/matrix memories. Its mLSTM branch makes the connection to matrix-valued associative memory especially clear.

### RWKV

Primary: https://arxiv.org/abs/2503.14456

RWKV is best treated as a recurrent-native architecture at the boundary with attention-like formulations. Its modern versions add more expressive dynamic state evolution and richer gating.

## Dominant research question

> How should a learned recurrent state be gated and structured so that useful information survives without making recurrence unstable or untrainable?

---

# 5. Lineage B — State Space Models (SSMs)

## Core idea

An SSM models a latent dynamical state:

```math
h_t = A h_{t-1} + B x_t,
```

```math
y_t = C h_t + D x_t.
```

The mathematical lineage comes from dynamical systems and control theory, even though modern SSMs can operate computationally like recurrent neural networks.

## Main memory strategy

**Control the dynamics of the state itself: stability, timescales, spectrum, and input-dependent selectivity.**

### S4

Primary: https://arxiv.org/abs/2111.00396

S4 made structured state-space dynamics practical for long-sequence deep learning.

### LRU as a bridge

Primary: https://arxiv.org/abs/2303.06349

LRU showed that much of the benefit associated with modern SSMs can be recovered by carefully designed linear recurrence: diagonalization, parameterization, initialization, and normalization.

This is one of the cleanest bridges between the RNN and SSM viewpoints.

### Mamba

Primary: https://arxiv.org/abs/2312.00752

Mamba introduces **selectivity**: parts of the state-space dynamics depend on the current input.

The conceptual shift is:

```text
fixed dynamical memory
        ->
input-conditioned dynamical memory
```

The model can therefore determine, based on the current token, what should enter or leave the state.

### Mamba-2 / Mamba-3

- Mamba-2 / SSD: https://proceedings.mlr.press/v235/dao24a.html
- Mamba-3: https://arxiv.org/abs/2603.15569

These continue the line through stronger algebraic links to attention, more expressive recurrence, state tracking, and hardware-efficient implementations.

## Dominant research question

> What state dynamics and timescale spectrum allow information to persist, transform, and disappear at the right rates?

This is different from the DeltaNet line: the SSM literature is primarily phrased in terms of **dynamics and selectivity**, not associative-memory editing.

---

# 6. Lineage C — recurrent linear attention as associative memory

This is currently the lineage in which the phrase **memory management** is most literal.

## Core idea

Linear attention can be written recurrently using a matrix-valued state. In a simplified form:

```math
S_t = S_{t-1} + \phi(k_t) v_t^\top,
```

```math
y_t = \phi(q_t)^\top S_t.
```

The state behaves like a finite associative memory relating keys to values.

The problem is therefore not only "how long does the state last?" but:

> How do we edit a finite memory without catastrophic interference between associations?

## 6.1 DeltaNet — targeted correction

Primary lineage paper: https://arxiv.org/abs/2406.06484

The delta rule writes according to prediction error rather than merely accumulating another outer product.

Conceptually:

```text
read current association
        ->
measure error
        ->
write a targeted correction
```

## 6.2 Gated DeltaNet — erase plus targeted write

Primary: https://openreview.net/forum?id=r8H7xhYPwz

Gated DeltaNet combines:

- global forgetting/gating;
- targeted delta-rule correction.

The memory problem becomes explicitly decomposable into **erase** and **write** behavior.

## 6.3 KDA / Kimi Linear — finer-grained gating

Primary: https://arxiv.org/abs/2510.26692

Kimi Delta Attention increases the granularity of state control and places the mechanism inside a large hybrid language model.

Important: Kimi Linear is not evidence that recurrence alone solves long-context recall. Its final architecture deliberately mixes recurrent KDA memory with explicit attention.

## 6.4 Gated DeltaNet-2 — decouple erase and write

Primary: https://arxiv.org/abs/2605.22791

A key refinement is:

```text
erase decision != write decision
```

The amount of old state removed and the amount of new information written are no longer forced to share the same control.

## 6.5 Kalman Delta Networks — add uncertainty

Primary: https://arxiv.org/abs/2609.07816

KDN adds a second object to memory:

```math
(S_t, P_t)
```

where:

- `S_t` is memory content;
- `P_t` represents uncertainty/confidence about that memory.

This changes the write question from:

> How large is the current prediction error?

to:

> How large is the error, and how confident am I in the state I am about to overwrite?

KDN is a good example of lineages mathematically cross-pollinating: it evolves DeltaNet/GDN, but interprets the update through Kalman filtering / state-space estimation.

## 6.6 CCQ — improve the read operation

Primary: https://arxiv.org/abs/2606.01294

Curvature-Conditioned Query changes the **read/query** side rather than the write side.

Its central lesson is that memory quality depends not only on what is stored but on how the query interrogates a compressed state.

## 6.7 SpectralShift — recalibrate retention timescales

Primary: https://arxiv.org/abs/2609.14320

SpectralShift is applied to Gated DeltaNet but uses a characteristically SSM/dynamical-systems idea: reshape the spectrum of retention timescales when extending context.

This is a strong example of convergence between lineages.

## Dominant research question

> Given a finite matrix-valued memory, how should the model write, erase, read, and protect associations from interference?

A compact evolution is:

```text
linear attention
   -> targeted write (DeltaNet)
   -> gate + targeted write (GDN)
   -> finer gating (KDA)
   -> separate erase/write (GDN-2)
   -> uncertainty-aware write (KDN)
   -> better read dynamics (CCQ)
   -> calibrated retention spectrum (SpectralShift)
```

These mechanisms are **not generic RNN improvements**. Most are specific to recurrent linear-attention / associative-memory formulations.

---

# 7. Lineage D — hybrid recurrent + explicit-attention memory

## Motivation

A bounded recurrent state has a fundamental capacity problem:

```text
(x_1, ..., x_t) -> S_t
```

compresses a growing sequence into a fixed-size object.

Full attention instead retains an explicitly addressable collection of past representations.

This creates a recurring tradeoff:

```text
fixed recurrent state
    -> efficient, compressed memory

explicit attention/KV
    -> expensive, addressable memory
```

## Key evidence

### Zoology

Primary: https://arxiv.org/abs/2312.04927

Associative recall explains a substantial part of the gap between efficient recurrent/SSM-style models and attention in controlled settings.

### BASED

Primary: https://arxiv.org/abs/2402.18668

Makes the recall-throughput tradeoff explicit and combines linear attention with sliding-window attention.

### What Attention Recalls and Recurrence Controls

Primary: https://arxiv.org/abs/2609.04434

Causal interventions in hybrid language models provide an important functional decomposition:

- attention/KV is especially important for **exact addressable retrieval and bindings**;
- recurrent state can carry more **compressed semantic, behavioral, language, persona, or style information**.

This supports a strong current hypothesis:

> Attention and recurrence may be better understood as complementary memory channels rather than imperfect substitutes.

## Representative hybrids

- Griffin / RecurrentGemma: https://arxiv.org/abs/2402.19427
- Kimi Linear: https://arxiv.org/abs/2510.26692
- Jamba and other SSM-attention hybrids belong to the same broader design logic.

## Dominant research question

> Which information should be compressed into recurrent state, and which information should remain explicitly addressable?

This may be more important than asking whether recurrence or attention "wins" globally.

---

# 8. Lineage E — recurrent depth / loop-time memory

This is a separate memory problem from long-context token memory.

## Core idea

Repeated computation produces a sequence of internal states:

```math
z^{(1)}, z^{(2)}, ..., z^{(R)}.
```

The question becomes:

> Should previous reasoning/computation states remain accessible, and how should the model choose among them?

## RecurTrace / Loop Memory Attention

Primary: https://arxiv.org/abs/2609.03379

RecurTrace keeps a short history of previous loop states explicitly addressable and adds adaptive halting.

This is **loop-time memory**, not ordinary context memory.

## LSTM-UT

Primary: https://arxiv.org/abs/2609.19521

Provides an important caution: an expanding cache of old loop states can interfere with a corrected current state. Bounded gated memory performs better in the paper's synthetic recurrent-depth tasks.

The lesson is:

```text
more accessible history != automatically better memory
```

Selection, gating, compression, and stale-state suppression matter.

## Looped Flows

Primary: https://arxiv.org/abs/2609.11801

Attacks another part of the loop-time problem: **credit assignment and training across repeated computation**.

## Dominant research question

> How should intermediate computation be retained, revisited, gated, and trained across recurrent-depth iterations?

---

# 9. Cross-lineage methods

Some methods do not belong cleanly to one architecture family.

## Credit Stabilization through Time (CST)

Primary: https://arxiv.org/abs/2609.09157

CST targets the backward-pass problem of assigning useful credit to earlier recurrent states over long horizons.

This is closer to a **general recurrent-training principle** than to a specific Mamba, GDN, or LSTM memory mechanism.

## Systems/serving methods

Examples such as Tail-Replay and recurrent-state quantization address how recurrence behaves in deployed systems rather than redefining the memory update itself.

- Tail-Replay: https://arxiv.org/abs/2608.30310
- GDN 4-bit quantization study: https://arxiv.org/abs/2609.04098

---

# 10. What we should mean by "modern RNN" going forward

To avoid ambiguity, use **modern recurrent model** as the umbrella term.

Then specify the lineage whenever the mechanism matters:

```text
modern recurrent model
    |
    +-- RNN-native / gated recurrent architecture
    +-- SSM
    +-- recurrent linear attention / associative memory
    +-- recurrent-attention hybrid
    `-- recurrent-depth / loop-time model
```

Use **RNN** narrowly when discussing the classical neural-recurrence lineage, unless the conversation is intentionally operating at the higher-level recurrence abstraction.

This preserves the pedagogical progression that proved useful in the research process:

```text
general recurrence
    -> memory/state mechanism
    -> architectural families
    -> specific research lineages
    -> individual papers
```

---

# 11. Current memory-management map by lineage

| Lineage | Main object remembered | Dominant strategy | Representative work |
|---|---|---|---|
| RNN/LSTM | hidden/cell state | gates, dedicated memory structure | LSTM, xLSTM |
| SSM | latent dynamical state | stability, spectrum, timescales, selectivity | S4, Mamba |
| Linear attention | associative matrix state | write, erase, read, uncertainty | DeltaNet, GDN, KDA, GDN-2, KDN, CCQ |
| Hybrid | recurrent state + explicit KV/attention | divide compressed vs addressable memory | BASED, Griffin, Kimi Linear |
| Recurrent depth | prior computation/loop states | loop memory, gating, halting, credit assignment | RecurTrace, LSTM-UT, Looped Flows |
| Cross-lineage | training signal / deployment state | credit stabilization, reconstruction, quantization | CST, Tail-Replay |

---

# 12. Main conclusions as of 2026-09-19

1. **SSMs are not simply "better RNNs", but they are one of the main modern continuations of the recurrent idea in NLP.** Their lineage and mathematics come from state-space/dynamical-systems modeling.

2. **The modern recurrence renaissance has several distinct lineages.** Treating all of them as RNNs obscures what is actually improving.

3. **The SSM line is mainly improving state dynamics, selectivity, stability, and timescales.**

4. **The recurrent linear-attention line is currently the most explicit "memory-management" line.** It decomposes memory into write, erase, read, uncertainty, and retention behavior.

5. **Hybrid models increasingly look principled rather than temporary compromises.** Evidence suggests recurrent state and explicit attention can carry qualitatively different kinds of memory.

6. **Recurrent-depth memory is a separate problem from context memory.** RecurTrace-style loop memory should not be grouped with Mamba/GDN long-context memory without qualification.

7. **Fixed-state recurrence has a structural recall/capacity tradeoff.** Better state updates can reduce interference but do not automatically eliminate the information bottleneck created by compressing a growing history into a bounded state.

8. **More memory is not automatically better.** Explicitly preserving additional states can hurt if the model cannot select, gate, or discard stale information.

9. **The lineages are converging mathematically.** SSM spectral ideas appear in GDN context extension; Kalman/state-estimation ideas appear in DeltaNet; linear attention can be written recurrently; hybrids combine several mechanisms in one model.

10. The research question should therefore move from:

> "Are RNNs coming back?"

to:

> **Which recurrent lineage solves which part of the memory/computation problem, and what tradeoffs remain?**
