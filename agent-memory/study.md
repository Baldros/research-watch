# Agent Memory — Foundational Reading Guide

_Curated starting points, 2026-10-19. This is a manually maintained reading guide, not a scheduler output._

## Concepts before algorithms

1. **[Memory in the Age of AI Agents](https://arxiv.org/abs/2512.13564)** — Hu et al. (initial 2025, revised 2026). Broad **form / function / dynamics** framework. Start here to separate what a memory is from what it does and how it changes.
2. **[Rethinking Memory in AI: Taxonomy, Operations, Topics, and Future Directions](https://arxiv.org/abs/2505.00675)** — Du et al. (2025). Six operations: **consolidation, updating, indexing, forgetting, retrieval and compression**. Useful for designing and testing real lifecycle APIs.
3. **[Rethinking Memory Mechanisms of Foundation Agents in the Second Half](https://arxiv.org/abs/2602.06052)** — Huang et al. (2026). Distinguishes **internal/external substrate**, cognitive roles, and agent/user memory subjects.

## Method anchors by stream

| Stream | Foundation / method | Specific reading goal |
| --- | --- | --- |
| M1 | [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560) (Packer et al., 2023/24) | How a system moves information between context and persistent tiers; distinguish virtual context from trained model state. |
| M2 | [FadeMem](https://arxiv.org/abs/2601.18642) (Wei et al., 2026) | What decay scores control physically; compare forgetting to static retention and budgeted retrieval. |
| M2 | [Recurrent Harness for Agentic Continual Learning — ReHarness](https://arxiv.org/abs/2610.12668) (Xiao et al., 2026) | Backward revision of retrieved reflections under online distribution shift; model weights vs text edits. |
| M3 | [Memorilla: Latent Semantic Memory for LLMs](https://arxiv.org/abs/2610.21375) (Fallahpour et al., 2026) | Learned query-conditioned memory tokens from persistent embeddings; frozen backbone vs trainable memory module. |
| M3 | [Context Language Models](https://arxiv.org/abs/2609.37725) (Shao et al., 2026) | Compare zero-shot context file management, learned policies and RL: which parts actually train? |
| M3 (boundary) | [Titans: Learning to Memorize at Test Time](https://arxiv.org/abs/2501.00663) (2025) | Internal neural memory at test time; when does it count as agent-level persistent memory? |
| M4 | [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) (Shinn et al., 2023) | Episodic text reflections reused after task feedback without weight updates. |
| M4 / harness cross-tag | [Harness²: Recursive Agent Harnessing for an Open World](https://arxiv.org/abs/2610.14224) (Xu et al., 2026) | Task-level harness edits versus reusable cross-task improvement state; do not confuse with trained executor. |

## Questions to ask of every paper

- What **exact object** is persisted? File lines, rows, embeddings, tensors, recurrent state, learned weights, or generated skills?
- Is memory maintained in the **harness, auxiliary trainable module, or executor**? Which weights actually change?
- How are **writes, conflict resolution, provenance, deletion, restoration, retrieval and budget accounting** implemented?
- Is improvement due to better memory or simply more **history tokens, additional model calls, storage, fine-tuning or search**?
- Does it preserve accuracy across **distractors, updated facts, multi-session context, unfamiliar tasks and adversarial content**?

See [taxonomy.md](./taxonomy.md) for classification, and the four category research files for evolving evaluation evidence.
