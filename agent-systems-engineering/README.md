# Agent Systems Engineering Watch

This research stream monitors the engineering of production-grade AI agent systems as an end-to-end discipline rather than as isolated prompting or model-evaluation work.

## Scope

The monitoring program is organized around three connected layers:

1. **Harness engineering and system design** — context management, tool interfaces, memory, permissions, verification, planning, recovery, and runtime design.
2. **Loop, graph, and multi-agent engineering** — coordination topologies, decomposition, parallelism, single-vs-multi-agent tradeoffs, communication, shared state, evaluators, and stopping/retry policies.
3. **AgentOps and production operation** — observability, tracing, reliability, fault injection, durable execution, checkpointing, idempotency, side effects, incident response, root-cause analysis, evals, CI gates, canarying, rollback, versioning, cost, tokens, latency, sandboxing, human oversight, and continuous operation.

## Evidence policy

Academic papers, preprints, benchmarks, technical reports, white papers, engineering reports, production postmortems, experiments, and A/B tests may all be used when they contain concrete technical evidence.

A central goal is to compare claims against counterevidence. The research should explicitly identify when different organizations or papers reach different conclusions and examine whether the disagreement can be explained by differences in workload, benchmark, model, harness, scale, metrics, or environment.

Generic claims about commercial bias are not sufficient criticism on their own; the focus should remain on technical evidence.

## Research memory

`research.md` is the living memory for this research stream. Each scheduler run should read the current document before searching, then revise it in place rather than append a new weekly report.

The document should preserve primary-source links, evidence, metrics, limitations, counterevidence, and implications for agent-system engineering. Prior conclusions should be marked as still relevant, strengthened, weakened, contradicted, superseded, or deprioritized when the evidence changes.

Where useful, the document may maintain a contradiction matrix summarizing which sources agree or disagree on harness design, multi-agent systems, graph/loop architecture, or production operations. Useful historical material should be preserved in a compact archival section instead of silently deleted.
