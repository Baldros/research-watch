# Agent Systems Engineering Watch

This research stream monitors the engineering of production-grade AI agent systems as an end-to-end discipline rather than as isolated prompting or model-evaluation work.

## Scope

The monitoring program is organized around three connected layers:

1. **Harness engineering and system design** — context management, tool interfaces, memory, permissions, verification, planning, recovery, and runtime design.
2. **Loop, graph, and multi-agent engineering** — coordination topologies, decomposition, parallelism, single-vs-multi-agent tradeoffs, communication, shared state, evaluators, and stopping/retry policies.
3. **AgentOps and production operation** — observability, tracing, reliability, fault injection, durable execution, checkpointing, idempotency, side effects, incident response, root-cause analysis, evals, CI gates, canarying, rollback, versioning, cost, tokens, latency, sandboxing, human oversight, and continuous operation.

## Conceptual taxonomy

See [taxonomy.md](./taxonomy.md) for the four-way classification of harness engineering (H1), self-evolving harnesses (H2), evolver weight–harness co-evolution (H3), and executor–harness co-adaptation (H4), including decision rules and evidence standards. These classes describe different optimization mechanisms rather than a proven historical progression. The taxonomy is conceptual; empirical findings and changing assessments remain in `research.md`.

## Scheduler ownership and manual synthesis

Each of the four harness research categories has an independently maintained living evidence document:

| Category | Scheduler-owned research memory |
| --- | --- |
| H1 — Harness Engineering | [h1-harness-engineering/research.md](./h1-harness-engineering/research.md) |
| H2 — Self-Evolving Harness | [h2-self-evolving/research.md](./h2-self-evolving/research.md) |
| H3 — Weight–Harness Co-Evolution | [h3-weight-harness/research.md](./h3-weight-harness/research.md) |
| H4 — Executor–Harness Co-Adaptation | [h4-co-adaptation/research.md](./h4-co-adaptation/research.md) |

**Write boundary:** each scheduled watcher may update only its own category's `research.md`. The root [research.md](./research.md) is reserved for **human-authored cross-category synthesis**: it is not a scheduler output, must never be automatically updated, migrated, overwritten, or reorganized by the watchers, and retains its historical material on multi-agent orchestration and AgentOps. Watchers may read it as context and link to findings across categories. [taxonomy.md](./taxonomy.md) is also not automatically edited. Cross-category conclusions are for the human researcher to incorporate deliberately.

## Evidence policy

Academic papers, preprints, benchmarks, technical reports, white papers, engineering reports, production postmortems, experiments, and A/B tests may all be used when they contain concrete technical evidence.

A central goal is to compare claims against counterevidence. The research should explicitly identify when different organizations or papers reach different conclusions and examine whether the disagreement can be explained by differences in workload, benchmark, model, harness, scale, metrics, or environment.

Generic claims about commercial bias are not sufficient criticism on their own; the focus should remain on technical evidence.

## Research memory

`research.md` is the living memory for this research stream. Each scheduler run should read the current document before searching, then revise it in place rather than append a new weekly report.

The document should preserve primary-source links, evidence, metrics, limitations, counterevidence, and implications for agent-system engineering. Prior conclusions should be marked as still relevant, strengthened, weakened, contradicted, superseded, or deprioritized when the evidence changes.

Where useful, the document may maintain a contradiction matrix summarizing which sources agree or disagree on harness design, multi-agent systems, graph/loop architecture, or production operations. Useful historical material should be preserved in a compact archival section instead of silently deleted.
