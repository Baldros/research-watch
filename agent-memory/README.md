# Agent Memory Research Watch

A research program on **memory management in LLM-based agents**, covering persistent representation/retrieval, memory lifecycle, learned memory policies, and experiential/procedural learning. **Last structured: 2026-10-19.**

## Purpose and scope

The object of study is an agent's ability to **retain, selectively reuse, correct, and operationalize information over time**. Investigate what is stored, by whom, where it persists, how it is accessed or transformed, and how it changes downstream decisions. Examine model-internal state only when it demonstrably serves agent memory; generic KV-cache efficiency, RNN/SSM architecture, static RAG, and long-context benchmarks alone do not qualify.

**Core distinction:** a model can remain weight-frozen while an external memory changes. The update of a `.md` file, embedding index, episodic store or learned memory module is not automatically training of executor weights.

See [taxonomy.md](./taxonomy.md) for classification and decision rules; [study.md](./study.md) for stable survey and foundational reading.

## Four independently monitored streams

| ID | Research question | Scheduler-owned memory |
| --- | --- | --- |
| **M1 — Memory Systems & Retrieval** | How is agent memory represented, stored, addressed, and retrieved? | [m1-memory-systems/research.md](./m1-memory-systems/research.md) |
| **M2 — Memory Lifecycle & Evolution** | How are entries written, consolidated, corrected, and forgotten? | [m2-memory-lifecycle/research.md](./m2-memory-lifecycle/research.md) |
| **M3 — Learned Memory & Model–Memory Co-Adaptation** | What memory decisions/representations are learned, and which parameters are updated? | [m3-learned-memory/research.md](./m3-learned-memory/research.md) |
| **M4 — Experiential & Procedural Memory** | How do trajectories become transferable lessons, strategies and reusable skills? | [m4-experiential-memory/research.md](./m4-experiential-memory/research.md) |

### Strict write boundaries

- **Each scheduler owns exactly its M-category `research.md` and may update only that file.**
- [research.md](./research.md) is **manual, cross-category synthesis only**. All scheduled watchers may read it, but **none may create, replace, reorganize or update it**.
- `README.md`, `taxonomy.md`, `study.md`, and other categories' files are **read-only** to all schedulers. Changing this policy requires an explicit human decision.
- If GitHub permission is denied, write fails, or a PR requires approval: **stop and request explicit approval**; do not retry repeatedly, claim success, or substitute edits to unauthorized files.
- References to studies in another category are cross-links rather than duplicate studies; mark primary and secondary classifications.

## Evidence and assessment policy

Prefer versioned primary papers, peer-reviewed publications, public code, reproducible technical reports, documented production experiments and counterevidence. Record **title, authors, date, version/status, method, frozen vs trained components, persistence scope, memory substrate, operations, experimental setting, baselines, results, cost, failure modes and limitations**. Treat arXiv preprints as preliminary and distinguish tested effects from architecture proposals or vendor claims.

Required controls where applicable: **no-memory, full-history, naive RAG, rule-based memory, same-backbone and matched storage/context/token/compute budgets**, plus held-out tasks, long horizons, distribution shift, temporal updates, distractors, conflicting memories and multi-session evaluation. Track **accuracy, retrieval quality, retention, transfer, stale-memory harm, storage, latency and training/inference cost**. For personalization or shared memory also track consent, privacy, provenance, access controls and poisoning.

Maintain each `research.md` **in place as living synthesis**, not an append-only run log. First read the current record; add significant advances, update or retract weaker conclusions, preserve compact historical context, deduplicate papers and surface contradictions. If no meaningful change is found, avoid gratuitous edits and report that outcome.

## Interfaces with adjacent research streams

- [Agent Systems Engineering](../agent-systems-engineering/): harness and context engineering; M3 is **not** automatically H3/H4. Ordinary memory updates are not executor-weight adaptation.
- [RNN/SSM](../rnn-ssm/): mechanisms of recurrent/internal state, which enter this program only when tied to agent memory.
- [LLM Post-Training](../llm-post-training/): learned policies, RL, and neural module training.
- [AI Security](../ai-security/): memory poisoning, untrusted context, provenance, persistence of unsafe instructions and exfiltration.

## Scheduler response contract

Keep notifications short. State the strongest new result (or no material update), key evidence/limitation, and **link directly to the updated category file**. Put thorough technical reasoning in GitHub, not the chat summary.
