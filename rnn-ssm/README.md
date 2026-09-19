# RNN and SSM Advances

This research stream tracks modern recurrent sequence models, State Space Models (SSMs), linear-attention architectures, and recurrent/hybrid approaches used in NLP and large language models.

## Scope

Priority topics include:

- modern RNN architectures;
- State Space Models and selective state-space mechanisms;
- linear attention and recurrent attention formulations;
- hybrid recurrent-attention models;
- recurrent depth and looped computation when directly relevant;
- efficiency, long-context behavior, memory, training stability, and inference tradeoffs;
- model families and lines of work such as Mamba, RWKV, xLSTM, Gated DeltaNet, Kimi Linear, Griffin/RecurrentGemma, and related architectures.

## Research objective

The goal is to understand which recurrent ideas represent genuine architectural progress, which are mostly efficiency refinements, and how recurrent or state-space approaches compare with standard Transformer-style sequence processing.

The research should distinguish conceptual changes from incremental engineering improvements and explicitly connect new work to prior architectures when possible.

## Research memory

`research.md` is the living memory for this research stream. Each scheduler run should read the current state before searching and revise the document in place rather than create a new weekly report.

The document should preserve primary-source links, publication status, architectural relationships, benchmark context, experimental evidence, limitations, and unresolved questions. Prior papers or conclusions should be marked as still relevant, strengthened, weakened, contradicted, superseded, or deprioritized when new evidence changes the picture.

Useful historical material should remain available in a compact archival section instead of being silently deleted, while the main body should reflect the best current conceptual map of modern recurrent, state-space, and hybrid sequence architectures.

## Conceptual taxonomy

The project now separates the broad idea of modern recurrence into several research lineages: RNN-native architectures, State Space Models (SSMs), recurrent linear attention / associative memory, recurrent-attention hybrids, and recurrent-depth / loop-time models.

See [`taxonomy.md`](./taxonomy.md) for the durable conceptual map and the current classification of long-term-memory mechanisms. `research.md` remains the living evidence/frontier document and should be revised as new papers change those assessments.
