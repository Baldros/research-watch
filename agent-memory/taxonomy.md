# Agent Memory — Research Taxonomy M1–M4

_Last defined: 2026-10-19. Conceptual classification for research monitoring; not a proposed universal architecture or historical progression._

## 1. Central definitions and exclusions

**Agent memory**: information/state retained or reconstructed for an agent's later decisions across steps, tasks or sessions; can be textual, structured, vectorial, latent, or parametric. A **memory system** combines a substrate with operations (write/index/retrieve/update/consolidate/forget) and a controller. It may live in a harness, a trained auxiliary component, or the executor itself.

**Inference context** and KV-cache are normally transient; **memory persistence** must be separately demonstrated. Retrieving fixed external documents using generic RAG is not by itself persistent *agent* memory. **Reflection logs**, skills and accumulated experience can constitute memory even without neural parameter updates.

Ask first: **what is retained, where, for how long, who controls it, what changes as experience accumulates, and how was utility demonstrated?**

## 2. Primary research categories

| ID | Category | Primary problem | Inclusion examples | Typical non-examples |
| --- | --- | --- | --- | --- |
| **M1** | **Memory Systems & Retrieval** | Representation, persistence, indexing, memory hierarchies, retrieval and query-conditioned access | Persistent stores, episodic/vector/graph indexes, retrieval/routing protocols, paged memory, shared multi-agent memory with access semantics | Generic static document search without an agent-memory mechanism |
| **M2** | **Memory Lifecycle & Evolution** | Selective writing, merging, temporal updates, retroactive revision, conflict resolution, retention and forgetting | Importance/decay policies, consolidation, provenance-aware correction, obsolete-state removal | Pure top-k retrieval, unrelated context truncation |
| **M3** | **Learned Memory & Model–Memory Co-Adaptation** | Learned representations, reads/writes or memory-management policies; learning signals and trained components | Trainable memory modules, memory RL, learned latent retrieval, jointly optimized memory controller and executor | An LLM choosing files with prompts alone, unless the paper primarily establishes learning of that policy |
| **M4** | **Experiential & Procedural Memory** | Turning trajectories, failures or feedback into transferable lessons, plans and skills | Reflection-based episodic improvement, strategy/skill libraries, reusable procedural experience, continual agent adaptation | Logging events with no demonstrated subsequent reuse or transfer |

**Primary category is the paper's main technical contribution**, not the database format, the title's terminology or the fact that an agent uses memory. Add secondary category tags and explain the overlap.

### M2 versus M4 — explicit boundary

M2 asks **whether/how an existing memory record remains valid and useful over time** (retention, edit, merge, forget). M4 asks **how an experience is abstracted into knowledge or a procedure that improves future behavior**. ReHarness is primarily M2 due to backward revision of earlier reflections, with an M4 tag for experiential adaptation. Reflexion is M4 because feedback becomes reusable reflection to guide future attempts.

### M1 versus M3 — explicit boundary

M1 focuses on an architectural storage/retrieval substrate and non-learning access mechanisms. M3 requires a demonstrated learning component or policy (e.g., trained memory-token generator, memory-management RL). It must name the trained parameters: **executor, encoder, adapter, memory controller, evolver, or none**. Calling an LLM API or changing external memory does not imply parametric learning.

## 3. Orthogonal tags (record alongside M1–M4)

1. **Substrate / representation**: raw text/files, relational or graph records, vector embeddings, latent tokens/states, learned parametric/fast weights, hybrid.
2. **Memory function**: working, factual/semantic, episodic, procedural, perceptual/sensory, user-centric vs agent-centric.
3. **Atomic operations**: formation/write, indexing, retrieval, compression, updating, consolidation, revision, forgetting.
4. **Controller / updater**: deterministic harness rules, LLM-controlled tooling, separately trained controller, executor-learned policy, hybrid.
5. **Persistence**: single forward pass, recurrent continuation, within-task, inter-task, multi-session, durable cross-agent; specify reset and restoration semantics.
6. **Learning**: none, prompting/search, SFT, RL (name actual algorithm), representation learning, test-time training; distinguish module vs executor weights.
7. **Topology**: individual vs multi-agent shared memory, centralized vs distributed, scoped permissions and provenance.
8. **Evaluation**: workload and horizon, valid temporal facts, conflict/update test, held-out generalization, costs, safety and trust boundaries.

## 4. Classification procedure for new work

1. Verify a **real memory mechanism** rather than a memory metaphor.
2. Identify the **persistent state**, when it is written and its lifetime.
3. Trace the **read/write/update/forget path** down to actual files, records, tensors or weights when possible.
4. Identify **what is trained and what stays frozen**, and distinguish a learned rule from prompted model behavior.
5. State **the primary bottleneck** solved: access (M1), lifecycle integrity (M2), learned controller/representation (M3), or reusable experience (M4).
6. Assign **one primary category**, optional secondary tags, and the orthogonal tags above. If insufficiently specified, use **unclassified/claimed**, not a stronger inferred label.
7. Require counterevidence and budget-matched baselines before attributing performance improvements to memory.

**Examples:** MemGPT = M1 (M2 secondary); FadeMem = M2 (M1 secondary); ReHarness = M2 (M4 secondary); Memorilla = M3 (M1 secondary); CLM = M3 when its **learned policy** is the key contribution (M2 secondary); Titans = M3 only where neural memory adaptation is agent-relevant; Reflexion = M4 (M2 secondary); Harness² = M4 for procedural experience, cross-referenced to harness H2.

## 5. Evaluation standards

- **Correctness & utility:** downstream task success, factual fidelity, retrieval, handling temporally valid/contradictory facts, hallucinated memories, multi-hop memory and out-of-distribution transfer.
- **Causal attribution:** same executor and environment, compare no-memory/full-history/simple-RAG/static-summary baselines; ablate writes/reads/revision/forgetting/training separately.
- **Resources:** equal context tokens, memory size, training/search compute, read/write operations, latency and storage; include cost of constructing and maintaining memory.
- **Robustness:** varying horizon, repeated/interleaved sessions, adversarial or irrelevant experiences, changing user preferences, corrupted or poisoned memories, cross-agent permissions.
- **Quality labels:** controlled/matched evidence; preliminary setting; production report; proposal/unverified; weakened/contradicted. Avoid general claims from a single synthetic benchmark.

## 6. Research boundaries

Track the following **cross-cutting** concerns without turning them into new schedulers by default: safety (poisoning/privacy/provenance), evaluation and benchmarks, multi-agent sharing, and implementation/production costs.

A new paper may touch [harness engineering](../agent-systems-engineering/taxonomy.md), [RNN/SSM](../rnn-ssm/taxonomy.md), [post-training](../llm-post-training/) or [AI security](../ai-security/), but belongs primarily here only when its *memory contribution* is substantial. **M3 is not harness H3/H4**: those classifications depend specifically on model/harness parameter adaptation loops, not generic memory learning.

## 7. Research basis / entry points

- Hu et al., [Memory in the Age of AI Agents](https://arxiv.org/abs/2512.13564) (2025/updated 2026): forms, functions and dynamics.
- Du et al., [Rethinking Memory in AI](https://arxiv.org/abs/2505.00675) (2025): atomic memory operations and representation taxonomy.
- Huang et al., [Rethinking Memory Mechanisms of Foundation Agents in the Second Half](https://arxiv.org/abs/2602.06052) (2026): substrate, cognitive mechanism, and agent/user-centric dimensions.
- [MemGPT](https://arxiv.org/abs/2310.08560), [FadeMem](https://arxiv.org/abs/2601.18642), [ReHarness](https://arxiv.org/abs/2610.12668), [Memorilla](https://arxiv.org/abs/2610.21375), [CLM](https://arxiv.org/abs/2609.37725), [Reflexion](https://arxiv.org/abs/2303.11366): anchors, **not settled superiority claims**.

## 8. Scheduler protocol

Schedulers read this taxonomy and [README.md](./README.md) as **read-only** context, then edit **only their own category `research.md`**. Parent [research.md](./research.md) is **human-owned and never scheduler-edited**. Never silently reclassify historical materials across files, alter the taxonomy, or widen write permissions. Add cross-category links instead of synchronized edits.
