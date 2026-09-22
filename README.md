# Research Watch

A versioned research knowledge base for recurring literature-monitoring tasks run through ChatGPT schedulers.

The repository stores curated primary-source links, technical comparisons, evolving research conclusions, and longitudinal notes across several research programs. Each monitored topic has its own directory and README describing the scope and evidence policy.

## Research programs

| Area | Purpose |
| --- | --- |
| [Agent Systems Engineering](./agent-systems-engineering/) | Track end-to-end agent engineering, harness design, multi-agent/graph systems, and production AgentOps. |
| [Complexity-Guided Architecture](./complexity-guided-architecture/) | Track methods that relate measurable task/data complexity to neural-network architecture, especially width and depth. |
| [RNN and SSM Advances](./rnn-ssm/) | Track modern recurrent models, state-space models, linear attention, and hybrid sequence architectures. |
| [AI Security](./ai-security/) | Track research connecting model internals and training phenomena to system-level attack surfaces created by autonomy, tools, memory, and long-horizon execution. |
| [LLM Post-Training](./llm-post-training/) | Track post-training algorithms, reward/evaluator systems, agentic RL, synthetic-data loops, scaling, stability, and evidence isolating post-training gains from model/data/compute effects. |
| [Topological ML](./topological-ml/) | Track machine learning and deep learning methods that use topology, TDA, persistent homology, and higher-order structures. |
| [Mesh Generation](./mesh-generation/) | Track academic advances in mesh generation for CAE, CFD, FEM, and scientific simulation. |

## Repository model

Each topic directory maintains a single living research document:

```text
<topic>/
├── README.md
└── research.md
```

`README.md` defines what the research stream monitors. `research.md` is the persistent research memory maintained by the corresponding scheduler.

The research file is **not an append-only weekly log**. On each run, the scheduler should read the existing state, search for new evidence, and then revise the document so it reflects the best current understanding of the area. New papers may be added, prior conclusions may be strengthened or weakened, and older material may be marked as contradicted, superseded, or deprioritized when the evidence changes.

Useful historical context should be preserved rather than silently deleted. Superseded or lower-priority material can be moved into compact archival sections within the same document.

## Design goals

- **Persistence:** research links and conclusions survive beyond a single chat or scheduler run.
- **Traceability:** claims stay tied to primary sources and publication dates.
- **Deduplication:** recurring searches compare findings against the existing research memory before adding them.
- **Current-state synthesis:** each `research.md` should represent the present state of the research, not merely a chronology of searches.
- **Longitudinal memory:** changes in relevance, evidence strength, and technical disagreements remain visible over time.
- **Machine readability:** the repository remains simple enough to search, parse, embed, or incorporate into future RAG and research workflows.

## Automation

The repository is updated by recurring ChatGPT research schedulers. Each scheduler reads its topic README and existing `research.md`, performs a new literature search, reconciles new findings with the stored research state, and updates that same file in place.

The GitHub document is intentionally more technical and complete than the scheduler's chat response. Chat notifications should remain short and summarize only what materially changed, why it matters, and where to find the updated research document.
