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

Reports should distinguish between methods that merely correlate with architecture requirements and methods that provide stronger predictive, theoretical, or causal evidence.

## Output

Dated reports should emphasize:

- what is technically new;
- which complexity measure is used;
- how the measure relates to architecture, especially width and depth;
- whether the method is data-first, model-first, or jointly optimized;
- experimental and theoretical evidence;
- limitations and assumptions;
- how close the result comes to an interpretable architecture-selection strategy.

Primary sources should be preferred, and repeated papers should only reappear when there is a meaningful revision or new result.
