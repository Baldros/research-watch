# Agent Memory — Cross-Category Research Synthesis (MANUAL)

_Initialized 2026-10-19. This file is reserved for the human researcher; schedulers must **never write to it**._

## Purpose

Maintain a **manually integrated** conceptual picture of the relations and tensions between the four independent research streams. For their automatically maintained evidence, read [M1](./m1-memory-systems/research.md), [M2](./m2-memory-lifecycle/research.md), [M3](./m3-learned-memory/research.md), and [M4](./m4-experiential-memory/research.md).

## Initial synthesis — conceptual, not a tested composite system

| Research concern | Assigned stream | Candidate interface to other streams |
| --- | --- | --- |
| What and where to store; how to find it | M1 | M2 determines validity; M3 can train the retriever |
| Which memories to preserve, merge, edit or delete | M2 | M1 provides physical storage; M4 generates new lessons |
| How to learn memory representations and control policies | M3 | M1/M2 provide environment and actions; may be neural or harness-based |
| How to abstract experiences into reusable behaviors/skills | M4 | M2 maintains lessons, M1 recovers them, M3 may learn when to consult them |

These are **complementary research questions, not proven interoperable components**.

## Open questions for manual analysis

1. How to combine **forgetting and retroactive correction** without deleting rare but important evidence?
2. Can **learned retrieval** outperform carefully matched transparent retrieval under equal training, inference and storage budgets?
3. Which gains in experience-based systems survive **task/domain shifts**, rather than reflecting replay or benchmark overlap?
4. When memory is shared across agents, how do **provenance, authorization and contamination** change the design?
5. At what point does memory-policy optimization become genuine **model–memory co-adaptation**, and what is trained?
6. Can external files/records, latent memory and procedural skills coexist without duplicate or inconsistent knowledge?

## Manual update policy

- Read all four category documents and their primary sources before advancing any synthesis claim.
- Record disagreements and evidence quality; do not automatically infer a unified solution or chronology.
- Do not use this file for scheduler logs or copy-pasting each category's literature list.
- Changes here are **manual-only** and deliberately distinct from automatic research-memory maintenance.
