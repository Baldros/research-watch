# Continual & Test-Time Learning — Foundational Study Guide

_Curated on 2026-10-27. Human-owned reference document; the scheduler must not edit it._

## 1. Reading path

**Stage A — The classical problem.** Begin with catastrophic interference, stability–plasticity, and sequential learning protocols. Learn why adaptation to a new task can degrade earlier performance, and distinguish regularization, replay and architectural growth.

**Stage B — Adaptation at inference.** Read test-time training and TTT layers, then Titans, MIRAS and ATLAS. Trace *which parameters or states are updated*, their objective, whether the base model stays frozen, and the exact reset/persistence semantics.

**Stage C — Multiple learning timescales.** Read Nested Learning / Hope as an attempt to treat different model components and optimization processes as nested, differently paced learning problems. Keep theoretical framing separate from demonstrated forgetting resistance.

**Stage D — Durable consolidation.** Read Sleep and SEAL. Ask whether new knowledge survives repeated updates and changes to task distribution, and whether reported improvements survive compute- and capacity-matched controls.

## 2. Surveys and foundations

| Reference | Why read it |
| --- | --- |
| [McCloskey & Cohen (1989), *Catastrophic Interference in Connectionist Networks: The Sequential Learning Problem*](https://doi.org/10.1016/S0079-7421(08)60536-8) | Historical statement of interference during sequential training. |
| [Kirkpatrick et al. (2017), *Overcoming catastrophic forgetting in neural networks*](https://arxiv.org/abs/1612.00796) | Elastic Weight Consolidation (EWC): parameter-importance regularization. |
| [Parisi et al. (2019), *Continual Lifelong Learning with Neural Networks: A Review*](https://arxiv.org/abs/1802.07569) | Broad foundational CL landscape. |
| [Wang et al. (2024), *A Comprehensive Survey of Continual Learning: Theory, Method and Application*](https://arxiv.org/abs/2302.00487) | Taxonomies, mechanisms and evaluation. |
| [Shi et al., *Continual Learning of Large Language Models: A Comprehensive Survey*](https://doi.org/10.1145/3735633) | LLM-specific sequential adaptation, fine-tuning and forgetting. |
| [*From Static to Adaptive Inference: A Survey on Test-Time Learning*](https://doi.org/10.1111/exsy.70446) | Complementary survey of learning/adaptation during inference. |

## 3. Primary research anchors

| Axis | Paper | Specific question |
| --- | --- | --- |
| C2 | [Sun et al., *Test-Time Training with Self-Supervision for Generalization under Distribution Shifts*](https://arxiv.org/abs/1909.13231) | What changes at inference, with what self-supervised signal? |
| C2 | [Sun et al., *Learning to (Learn at Test Time): RNNs with Expressive Hidden States*](https://arxiv.org/abs/2407.04620) | How can a learner's parameters serve as recurrent state? |
| C2 | [Behrouz et al., *Titans: Learning to Memorize at Test Time*](https://arxiv.org/abs/2501.00663) | Fast neural memory, surprise, momentum, forgetting; state versus backbone updates. |
| C2 | [*It's All Connected: A Journey Through Test-Time Memorization, Attentional Bias, Retention, and Online Optimization* (MIRAS)](https://arxiv.org/abs/2504.13173) | Which general memory components govern objectives and update rules? |
| C2 | [*ATLAS: Learning to Optimally Memorize the Context at Test Time*](https://arxiv.org/abs/2505.23735) | Do improved online-memory objectives outperform simpler methods at matched budgets? |
| C3 | [*Nested Learning: The Illusion of Deep Learning Architectures*](https://arxiv.org/abs/2512.24695) | Learning as nested optimization, continuum memory and multiple update frequencies. Verify versions/dates rather than infer chronology from an arXiv number. |
| C4 | [*Language Models Need Sleep: Learning to Self-Modify and Consolidate Memories*](https://arxiv.org/abs/2606.03979) | Offline consolidation and synthetic experiences; distinguish durable weight updates from ephemeral neural state. |
| C4 | [*SEAL: Self-Adapting Language Models*](https://arxiv.org/search/?query=SEAL%3A+Self-Adapting+Language+Models&searchtype=all) | Self-generated updates and persistent adaptation; measure forgetting across sequential cycles. |

The papers above are **anchors, not proof of a single end-to-end architecture**. Check the latest version, author list, publication status and available independent replications on each run. A paper on better long-context recall alone does not establish continual learning.

## 4. Questions to take into each reading

1. What changes: input context, recurrent state, fast memory-module weights, adapters, main backbone, optimizer state, or an external store?
2. When are updates applied: per token, per segment, per query, between sessions, during offline consolidation, or across tasks?
3. What resets? Does new knowledge remain after a model checkpoint reload?
4. Which objective drives adaptation? Gradient descent, learned optimizer, self-supervised task, RL, distillation, replay?
5. How much pre-existing competence is retained after **multiple** updates; how are forgetting and forward transfer measured?
6. What is the true cost versus a frozen model with longer context, retrieval, ordinary continued training, or simpler memory?
7. Does the experiment demonstrate *durable adaptation* or only *temporary within-stream recall*?

## 5. Connections to the rest of the repository

- [Agent Memory / M3](../agent-memory/m3-learned-memory/research.md): learned *agent* memory and its trainable controllers; Titans is mostly a boundary reference until agent-memory benefits are demonstrated.
- [RNN/SSM](../rnn-ssm/): recurrent hidden states and associative-memory architectures.
- [LLM Post-Training](../llm-post-training/): training procedures that may be used for consolidation, but not all post-training constitutes continual learning.
- [AI Security](../ai-security/): unsafe or poisoned persistent adaptation.

See [README.md](./README.md) for scope and [research.md](./research.md) for the continuously updated evidence synthesis.
