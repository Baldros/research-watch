# Topological ML — Living Research Map

_Last reviewed: 2026-10-01_

This is a curated map of machine learning that uses topology as representation, architecture, constraint, statistical object, or generative validity condition.

## Executive view

The main change since 2026-09-12 is a shift from **topology as an auxiliary descriptor** toward **topology as the computational substrate of the model**.

The strongest new scientific-ML result is **Dynamic Kuramoto–Hodge Operators (DKHO)**, where fields stay on vertices/edges/faces, boundary and coboundary operators constrain information flow, and learned dynamics adapts coupling to each PDE instance. **T-SNN** adds strong evidence that higher-order structure can evolve over time. **CNMTDL** combines Hodge decomposition with combinatorial attention. **PHL/MFPHL** extends persistent topology to directed many-body interactions while reducing spectral cost with matrix-free stochastic trace estimation. **TopoEmbedX** is mainly infrastructure, but makes systematic comparison across topological domains easier.

For mesh generation and simulation, the working thesis is strengthened: the promising direction is not “persistent-homology features -> generic network -> mesh,” but **topological/cochain state + geometric residual + physics + learned policy/operator + topology-valid numerical kernel**.

---

## 1. Scientific ML: topology as operator structure

### Dynamic Kuramoto–Hodge Operators for PDEs on Complex Geometries and Topologies — Li & Song
- Primary: https://arxiv.org/abs/2609.33693
- Submitted: 2026-09-27; updated: 2026-09-29.
- **Status: new / highest priority.**
- Conditions live on native cochain supports; boundary/coboundary operators form the Dirac structure; harmonic and non-harmonic Hodge components are decoded separately.
- On porous-medium Darcy flow, torus transport-diffusion, and cavity magnetostatics, the authors report about **61% average error reduction** for DKHO-large relative to leading baselines. DKHO-small remains competitive with **11.5–24.3%** as many parameters.
- **Why it matters:** topology determines where information may flow, while learning determines adaptive coupling. This is much stronger than appending topological features to a generic neural operator.
- **Limitations:** preprint only; limited benchmark breadth; needs independent reproduction and tests on larger industrial meshes.
- **Mesh relevance:** very high. FEM/FVM fields naturally live on cells of different ranks, so collapsing everything to an untyped graph may discard useful structure.

### Sparsely connected neural network representation of Lagrange finite element function — Hao, Huang & Yi
- Primary: https://arxiv.org/abs/2609.23299
- Submitted: 2026-09-20.
- **Status: new / adjacent but strategically relevant.**
- Constructs sparse neural networks exactly matching arbitrary-order Lagrange finite-element spaces on simplicial meshes.
- Demonstrates interpolation across non-matching meshes and an adaptive-FEM application.
- **Importance:** shows that classical discretization structure can be embedded directly into a neural computational graph instead of approximated by a black box.
- **Limitation:** not TDL in the usual sense and does not learn topology.

**Assessment:** the earlier “higher-order encoder + classical numerical kernel” thesis is **strengthened**. DKHO suggests that topology can define the operator algebra itself.

---

## 2. Dynamic and attention-based higher-order models

### T-SNN: Temporal Simplicial Neural Network for EEG Decoding — Malik et al.
- Primary: https://arxiv.org/abs/2609.34002
- Submitted: 2026-09-27.
- **Status: new / important architectural evidence.**
- Represents consecutive time windows as evolving simplicial complexes; simplicial convolutions model within-window structure and recurrent states carry cell-specific history.
- On SEED-VII, reported EEG-only accuracy is **70.94% trial-wise** and **69.87% leave-one-subject-out**, versus about 56% and 40% for the paper's Transformer baseline.
- **Importance:** higher-order topology need not be static; cell identities can appear/disappear while temporal state persists.
- **Limitations:** one main application/benchmark; higher-order simplices are built by clique completion from thresholded pairwise correlations.
- **Mesh relevance:** strong analogy to adaptive remeshing and moving-interface problems.

### Combinatorial Network-Based Manifold Topological Deep Learning for Image Analysis — Wachira et al.
- Primary: https://arxiv.org/abs/2609.25453
- Submitted: 2026-09-21.
- **Status: new / medium-high priority.**
- Combines Hodge decomposition with combinatorial attention across cell ranks on discrete manifold representations.
- Evaluated on six 2D/3D MedMNIST v2 datasets.
- **Importance:** supports the pattern **topological decomposition first, learned attention second**.
- **Limitations:** medical imaging is far from solver-grade geometry; stronger ablations are needed to isolate the source of gains.
- **Mesh relevance:** moderate-to-high because the same construction maps naturally to cell-complex meshes.

**Assessment:** the claim that higher-order structure should be architectural, not merely a feature vector, is **strengthened**. The field is also moving toward dynamic and attention-based higher-order models.

---

## 3. Persistent topology beyond pairwise undirected complexes

### PHL: Persistent Hyperdigraph Learning for Protein-Protein Binding Affinity Prediction — Xu, Wang & Chen
- Primary: https://arxiv.org/abs/2609.33122
- Submitted: 2026-09-27.
- **Status: new / high methodological interest.**
- Extends persistent-topological descriptors to directed many-body hyperdigraph interactions.
- Introduces MFPHL, using stochastic trace estimation over sparse boundary operators instead of full spectral decomposition.
- Reported feature-generation time is reduced by roughly **two orders of magnitude**. Accuracy matches spectral PHL within seed variability on one benchmark but is lower on two larger datasets.
- **Importance:** attacks two real weaknesses of topology-based ML: pairwise/undirected simplification and spectral cost.
- **Mesh relevance:** matrix-free Hodge/persistent summaries may be practical as diagnostics or critics on large meshes.

### Filtered groupoid persistence — Alsulami
- Primary: https://doi.org/10.3934/math.20261180
- Published: 2026-09-14, *AIMS Mathematics*.
- **Status: new peer-reviewed theory.**
- Incorporates finite-group symmetries/equivalence relations into persistence via filtered action groupoids.
- **Importance:** “topological compression” need not ignore known physical symmetries; symmetries can be part of the invariant.
- **Limitations:** mathematical framework, not yet a neural architecture or scalable ML result.
- **Mesh relevance:** potentially useful for periodic domains, repeated components, and quotient/symmetry structure.

---

## 4. Representation-learning infrastructure

### TopoEmbedX — Frantzen et al.
- Primary: https://arxiv.org/abs/2609.37884
- Submitted: 2026-09-29; updated: 2026-09-30.
- **Status: new / infrastructure.**
- Unifies DeepCell, Cell2Vec, CellDiff2Vec, HOLE, and HOGLEE and adds ComplexNetMF, ComplexRep, ComplexRandNE, ComplexWalklets, and ComplexHeat.
- Uses an augmented Hasse graph to extend familiar embedding techniques to higher-order domains.
- **Importance:** improves apples-to-apples comparison across simplicial, cellular, and hypergraph representations.
- **Limitation:** reducing a complex to a Hasse graph can still hide orientation/Hodge semantics if the downstream model treats it as an ordinary graph.
- **Mesh relevance:** useful baseline for testing whether explicitly higher-order representations outperform graph reductions.

---

## 5. Prior anchors — current status

- **Geodesic-informed generative diffusion** — https://arxiv.org/abs/2609.08153 — **still relevant.** Strong evidence for topology-preserving generative parameterizations when topology should remain fixed; too restrictive for genuine topology change.
- **Topologically consistent parcel vectorization** — https://arxiv.org/abs/2609.07520 — **still relevant / strengthened as an architecture pattern.** Learned geometric cues plus deterministic shared-graph reconstruction remain highly relevant to meshing.
- **Spatiotemporal Persistence Landscapes** — https://link.springer.com/article/10.1007/s41468-026-00251-1 — **still relevant.** Now complements T-SNN: one gives statistical summaries of evolving topology, the other learns directly on evolving complexes.
- **Topology Obstructs Pure Foundation Neural Quantum States** — https://arxiv.org/abs/2609.07591 — **still high conceptual importance.** Representation families can be topologically incapable before optimization begins.
- **Hodge fixed-index robustness** — https://journals.aps.org/pre/abstract/10.1103/n4ps-wmd7 — **still relevant.** Supports the claim that the 1-skeleton can miss higher-order failures.
- **Periodic Topological Deep Learning for Polymer Design** — https://arxiv.org/abs/2605.26833 — **still a strong scientific application anchor.**
- **COSMOS / Continuous Simplicial Neural Networks** — https://arxiv.org/abs/2503.12919 — **still an architecture anchor.**
- **Topological Autoencoders / ++** — https://arxiv.org/abs/1906.00722 and https://arxiv.org/abs/2502.20215 — **foundational; generic “topological loss” interpretation remains weakened.**

---

## 6. Mesh-generation / scientific-geometry synthesis

A useful factorization is:

1. **Topological/combinatorial state:** oriented incidence, cells, boundary/coboundary maps, homology/cohomology, Hodge/Dirac operators.
2. **Geometric residual:** coordinates, metric, curvature, anisotropy, sharp features, local feature size.
3. **Physical state:** PDE variables, boundary conditions, constitutive parameters, error indicators, solver residuals.
4. **Learned policy/operator:** refinement/coarsening, movement, decomposition, topology-valid local moves, or cochain-native PDE updates.
5. **Validity mechanism:** simplicial/cellular constraints, manifold/link checks, diffeomorphic maps when appropriate, classical meshing kernels, exact FE structure.
6. **Evaluation:** solver error/runtime, geometric element quality, topology diagnostics, and uncertainty calibration.

### Current research hypothesis
The highest-value form of topological DL for meshing is to make the discretization's **topological algebra part of the architecture and action space**, while learning the geometry- and physics-dependent decisions.

A concise principle is:

> **Topology determines admissible structure and information flow; geometry supplies metric detail; physics supplies task relevance; learning chooses among admissible decisions.**

DKHO materially strengthens this hypothesis.

### High-value open questions
- Can cochain/simplicial mesh policies outperform graph encoders when the 1-skeleton is fixed but face/cell incidence differs?
- Can a learned mesher use topology-valid local moves as its action vocabulary?
- Can Hodge/Dirac structure identify regions where global harmonic information matters?
- Can matrix-free topological summaries scale as critics to million-cell meshes?
- Can dynamic simplicial state tracking transfer to adaptive remeshing?
- What is the minimal geometric residual needed once topology/combinatorics are explicit?
- How should controlled topology change be represented when diffeomorphic parameterizations are too restrictive?

---

## 7. Revised interpretations

- **“PH features alone make a model topological.” — Deprioritized.** New work increasingly places topology in operator algebra, message-passing support, action spaces, or hard validity.
- **“Mesh = graph is a sufficient generic abstraction.” — Further weakened.** New cochain-native PDE operators and simplicial temporal models exploit information beyond the 1-skeleton.
- **“Topological structure must be static.” — Weakened.** Spatiotemporal persistence and T-SNN treat changing structure explicitly.
- **“Higher-order topology is necessarily too expensive.” — Weakened, not disproven.** MFPHL shows large speedups are possible, but accuracy trade-offs remain.
- **“A topology-aware loss guarantees validity.” — Weakened.** Hard parameterizations and topology-valid reconstruction remain more convincing when validity is non-negotiable.
- **“Topology preservation is always desirable.” — False.** Fracture, merging interfaces, remeshing with connectivity change, and variable-genus generation require controlled topology change.
