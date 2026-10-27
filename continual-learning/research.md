# Continual & Test-Time Learning — Living Research State

_Initialized: 2026-10-27. Baseline literature map based on published research lines; this is **not** a completed independent benchmark audit. The scheduled watcher owns only this file._

## Current synthesis

1. **Continual learning is an established research problem, not a 2025 invention.** Sequential learning causes interference and forgetting. Replay, regularization (e.g., EWC), architectural isolation/expansion and distillation address different pieces of the stability–plasticity tradeoff; none should be presented as universally solving lifelong learning.
2. **Test-time learning and lifelong learning are not equivalent.** Online updates to a recurrent or fast-weight memory can improve *within-stream* adaptation, but durability across tasks and sessions depends on state retention, reset behavior and consolidation. Report the lifetime of each adapted tensor.
3. **Titans → MIRAS → ATLAS is a useful technical lineage, not a proof of perpetual learning.** The publications study trainable neural memory and online optimization, evolving from particular update mechanisms toward broader formulations and stronger memory objectives. Report compute, memory size, long-context controls and independent replication.
4. **Nested Learning / Hope raises the central multi-timescale question:** whether distinct components or optimization levels can learn and retain information at different rates. Treat the conceptual framework and experimentally validated lifelong-retention outcomes separately.
5. **Sleep examines a different bottleneck: consolidation.** Transferring transient knowledge into more durable parameters via distillation/self-generated experiences is a plausible bridge from temporary adaptation to continual learning; benchmark-specific improvement and synthetic recall do not establish unlimited autonomous adaptation.
6. **There is no demonstrated universal synthesis across C1–C4 in this repository yet.** Any end-to-end 'fast memory → consolidation → stable competence' diagram is a research hypothesis unless supported by controlled experiments.

## Primary research map

| ID | Axis | Anchor | Main mechanism or open test | Status of evidence in this file |
| --- | --- | --- | --- | --- |
| F1 | C1 | [McCloskey & Cohen (1989)](https://doi.org/10.1016/S0079-7421(08)60536-8) | Sequential interference | Historical foundation |
| F2 | C1 | [Kirkpatrick et al., EWC (2017)](https://arxiv.org/abs/1612.00796) | Protect important weights under task changes | Foundational primary study |
| T1 | C2 | [Test-Time Training with Self-Supervision](https://arxiv.org/abs/1909.13231) | Adapt on unlabeled target samples | Precursor; not agent memory |
| T2 | C2 | [Learning to (Learn at Test Time)](https://arxiv.org/abs/2407.04620) | Neural weights as recurrent state | TTT architectural precursor |
| T3 | C2 | [Titans](https://arxiv.org/abs/2501.00663) | Surprise-driven memory updates, momentum and forgetting | Author-reported long-context/adaptation evidence; verify persistence |
| T4 | C2 | [MIRAS](https://arxiv.org/abs/2504.13173) | Generalized online-memory design space | Author-reported; examine controlled ablations |
| T5 | C2 | [ATLAS](https://arxiv.org/abs/2505.23735) | More expressive test-time memorization objectives | Author-reported; compare update compute |
| N1 | C3 | [Nested Learning / Hope](https://arxiv.org/abs/2512.24695) | Nested optimization and multi-frequency memory | Distinguish formal framework from evidence |
| P1 | C4 | [Language Models Need Sleep](https://arxiv.org/abs/2606.03979) | Offline/periodic knowledge consolidation, synthetic training | Preprint; durable retention under repeated updates remains essential to audit |
| P2 | C4 | [SEAL: Self-Adapting Language Models](https://arxiv.org/search/?query=SEAL%3A+Self-Adapting+Language+Models&searchtype=all) | Model-generated self-update data | Inspect sequential-forgetting limitations |

## Evaluation matrix for new literature

| Claim | Minimum validation | Critical confound |
| --- | --- | --- |
| Learns continuously | Performance on old and new tasks over repeated sequential updates; forward/backward transfer | Mere adaptation on one test stream |
| Remembers durably | Persistence after restart/reset/checkpoint and later unrelated tasks | Temporary recurrent state or context |
| Resists forgetting | Task-agnostic and nonstationary streams; retention-versus-plasticity plots | Repeated evaluation on known training tasks |
| Consolidates knowledge | Isolate transfer from transient to persistent parameters and assess destructive interference | Extra model capacity, data, replay or offline compute |
| More efficient than alternatives | Matched tokens, storage, model size, gradient steps, rollouts and latency | Hidden adaptation cost |
| Robust autonomous updating | Rollback, conflicting facts, adversarial data and evaluation after many cycles | Cherry-picked successes, contaminated self-generated data |

## Priority unresolved questions

1. Does an online neural-memory mechanism yield **cross-session / cross-task** retention after independent task streams and explicit resets?
2. Do multi-timescale updates outperform replay, adapters, plain continued pretraining or carefully budgeted retrieval at **matched total cost**?
3. Can consolidation avoid erasing rare but important knowledge, especially as model-generated training data accumulates?
4. Which components actually change: recurrent state, learned fast weights, memory adapters, main model weights, or all of these?
5. Are gains caused by algorithmic innovation or simply larger memory/extra compute?
6. Can memory modification be made safe, reversible, auditable and resilient to poisoning?

## Cross-stream ownership

- [Agent Memory M3](../agent-memory/m3-learned-memory/research.md): agent-specific learned memory policy/representation. Cross-link rather than duplicate agent papers.
- [RNN/SSM](../rnn-ssm/): recurrent or associative architectures without a core continual-adaptation claim.
- [Post-Training](../llm-post-training/): general SFT/RL/distillation optimizers without sequential-retention objectives.
- [AI Security](../ai-security/): persistent model contamination and adaptation risks.

## Maintenance protocol

On each run, review [README.md](./README.md), [study.md](./study.md) and this file. Verify primary sources and version dates; perform focused search for new work, revisions, replications, stronger baselines and negative results. Modify **only** this document to reconcile evidence, not merely append papers. Preserve compact history and identify claims as strengthened, weakened, contradicted or superseded. Do not edit the manual reference files or any adjacent stream. On GitHub approval/permission failures, stop and ask the user expressly; no retries. If no meaningful new findings, keep this file unchanged and do not notify.

## History

- **2026-10-27:** Research area opened with classical CL, TTT/neural memory, multi-timescale learning and parametric consolidation as four monitoring axes. Initial list is an **anchor map**, not independently validated comparative evidence.
