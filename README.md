# Research Watch

A versioned research knowledge base for recurring literature-monitoring tasks run through ChatGPT schedulers.

The repository stores curated research reports, primary-source links, technical comparisons, and longitudinal notes across several research programs. Each monitored topic has its own directory and README describing the scope, selection criteria, and intended report structure.

## Research programs

| Area | Purpose |
| --- | --- |
| [Agent Systems Engineering](./agent-systems-engineering/) | Track end-to-end agent engineering, harness design, multi-agent/graph systems, and production AgentOps. |
| [Complexity-Guided Architecture](./complexity-guided-architecture/) | Track methods that relate measurable task/data complexity to neural-network architecture, especially width and depth. |
| [RNN and SSM Advances](./rnn-ssm/) | Track modern recurrent models, state-space models, linear attention, and hybrid sequence architectures. |
| [AI Security](./ai-security/) | Track research connecting model internals and training phenomena to system-level attack surfaces created by autonomy, tools, memory, and long-horizon execution. |
| [Topological ML](./topological-ml/) | Track machine learning and deep learning methods that use topology, TDA, persistent homology, and higher-order structures. |
| [Mesh Generation](./mesh-generation/) | Track academic advances in mesh generation for CAE, CFD, FEM, and scientific simulation. |

## Repository model

Each topic directory is designed to contain dated research reports produced by its corresponding scheduler. The intended convention is:

```text
<topic>/
├── README.md
└── reports/
    ├── YYYY-MM-DD.md
    └── ...
```

Reports should favor primary sources, preserve direct links to papers and technical reports, distinguish evidence from speculation, and avoid repeating previously cataloged work unless there is a meaningful revision or new result.

## Design goals

- **Persistence:** research links and conclusions should survive beyond a single chat or scheduler run.
- **Traceability:** claims should remain tied to primary sources and dated reports.
- **Deduplication:** recurring searches should be able to compare new findings against the existing repository.
- **Longitudinal analysis:** reports should make it possible to see how a research area, claim, or technical disagreement evolves over time.
- **Machine readability:** the repository should remain simple enough to be searched, parsed, embedded, or incorporated into future RAG/research workflows.

## Automation

The repository is intended to be updated by recurring ChatGPT research schedulers. A scheduler may read existing reports to detect duplicates and prior conclusions, perform a new literature search, and then create a new dated report containing only meaningful additions or revisions.
