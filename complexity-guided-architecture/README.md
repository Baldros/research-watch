# Complexity-Guided Architecture

This research stream monitors work on selecting or constraining neural-network architecture from measurable properties of the task, dataset, or learned representation.

## Core question

The guiding question is whether interpretable measures of data or task complexity can help constrain architectural choices such as network width and depth, reducing reliance on blind search or purely empirical scaling.

## Scope

Priority areas include:

- topology and Topological Data Analysis (TDA);
- geometry of data and learned representations;
- intrinsic dimension;
- information theory and entropy-based measures;
- Fisher information and effective dimension;
- compositional complexity;
- data-first architecture selection;
- task embeddings;
- dataset-to-architecture prediction;
- meta-NAS and related architecture-selection methods.

## Research objective

The goal is not to assume that a single scalar complexity measure can directly determine an optimal architecture. Instead, the monitoring program looks for evidence that multiple measurable properties can progressively constrain the viable architecture space.

The research should distinguish between methods that merely correlate with architecture requirements and methods that provide stronger predictive, theoretical, or causal evidence.

## Research memory

`research.md` is the living memory for this research stream. Each scheduler run should read it first, then revise it in place rather than create a dated weekly report.

The document should track what is technically new, which complexity measures are used, how they relate to architecture—especially width and depth—and whether the evidence is empirical, theoretical, predictive, or causal. Prior ideas and papers should be marked as still relevant, strengthened, weakened, contradicted, superseded, or deprioritized when appropriate.

Primary-source links, assumptions, limitations, and the connection to the central architecture-selection thesis should be preserved. Useful older material should remain available in a compact archival section rather than be silently removed.
