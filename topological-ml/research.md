# Topological ML — Living Research Map

_Last reviewed: 2026-09-12_

This document is a curated research memory for machine learning methods that use topology as a representation, architectural primitive, learning constraint, statistical object, or generative validity condition. It is organized by conceptual role rather than by scheduler chronology.

## Executive view

The most important current distinction is between **topology as a descriptor** and **topology as a structural constraint on the hypothesis space**.

- Persistent homology used only as an auxiliary feature remains useful, but usually yields incremental gains unless it captures structure unavailable to the base representation.
- Higher-order architectures based on simplicial/cell complexes remain the clearest route when the data natively has vertices, edges, faces, cells, or many-body interactions.
- The strongest new conceptual direction is to make topology part of the **admissible representation or transformation space**, so invalid outputs are impossible or structurally disfavored rather than repaired by a soft loss after generation.
- For scientific geometry and meshing, the most promising target architecture remains: **higher-order geometric/topological encoder + learned global policy/generator + topology-valid geometric action space + classical numerical/meshing kernel + solver feedback**.

No genuinely new simplicial/cellular neural architecture appeared in the 2026-09-06 to 2026-09-12 window. The important changes this week were instead in: (1) topology-preserving generative spaces, (2) statistical treatment of dynamic/persistent topological summaries, (3) topological obstructions to neural representation classes, and (4) stronger evidence that graph-level connectivity can miss higher-order failures.

---

## 1. Topology as a hard generative constraint

### Geodesic-informed Generative Diffusion Model for Topology-preserved Image Video Generation — Wu et al.
- **Primary source:** https://arxiv.org/abs/2609.08153
- **arXiv date:** 2026-09-08.
- **Status:** **Relevant, but not wholly new.** The arXiv posting is new this week, while the paper is an extended version of prior IGG work and the journal version is dated August 2026.
- **Construction:** The model does not diffuse directly in image intensity space. It learns a latent space of geodesic deformation trajectories and generates smooth invertible diffeomorphisms from a template.
- **Why it matters:** This moves topology preservation from a soft objective into the **parameterization of the generative process**. If the generated transformations remain diffeomorphic, topological type is preserved by construction.
- **Evidence:** Experiments include plant growth and brain MRI data; the authors compare against text/video generative baselines and report better topological consistency, while also using generated image-label pairs for downstream segmentation.
- **Limitations:** This only covers deformation within a fixed topological class. It cannot naturally create or destroy components, handles, holes, or branches when the task genuinely requires a topology change. The paper therefore supports a topology-preserving generator, not a universal topology-generating model.
- **Mesh / geometry connection:** Very strong for mesh morphing, ALE-style moving meshes, anatomical simulation, shape deformation, and topology-preserving remeshing. The transferable principle is to generate **valid transformations of an existing discretization**, not arbitrary coordinates/connectivity.

### Topologically Consistent Agricultural Parcel Vectorization with Semantic-Guided Diffusion and Topology-Aware Polygonization — Jiao et al.
- **Primary source:** https://arxiv.org/abs/2609.07520
- **Date:** 2026-09-07.
- **Status:** **New and relevant.**
- **Construction:** Diffusion predicts edge/vertex primitives; polygon faces are then reconstructed from a **shared planar graph** so neighboring parcels reuse common boundaries.
- **Why it matters:** The key insight is architectural separation: a neural generator proposes geometric primitives, while a deterministic topology-aware reconstruction stage enforces consistency among adjacent objects.
- **Evidence:** The paper reports strong parcel-vectorization performance, zero measured intrusion ratio, and the highest shared-edge recall on its evaluated datasets.
- **Limitations:** Domain-specific geospatial vectorization; topology is planar-graph consistency rather than homology or higher-order TDL. Generalization to 3D meshes, non-manifold geometry, or solver-grade discretizations is unproven.
- **Mesh / geometry connection:** Highly relevant pattern for meshing: **learn geometry cues, reconstruct connectivity with a topology-valid kernel**. This is more credible for CAE than unconstrained end-to-end generation of all mesh connectivity.

### Assessment update
The evidence now favors **hard or semi-hard structural validity** over topology only as a penalty term for generative geometry. Soft topological losses remain useful, but current examples increasingly show that topology can be encoded in the transformation family or reconstruction kernel itself.

---

## 2. Dynamic topology and topology for scientific simulation

### Spatiotemporal Persistence Landscapes — Flammer & Hüper
- **Primary source:** https://link.springer.com/article/10.1007/s41468-026-00251-1
- **Version of record:** 2026-09-08, Journal of Applied and Computational Topology.
- **Status:** **New publication and conceptually important.**
- **Construction:** Combines zigzag persistence and multiparameter ideas to represent structures persistent simultaneously across **time and spatial scale**. The resulting landscapes live in Banach/Lebesgue function spaces and have a stability result with respect to an adapted interleaving distance.
- **Why it matters:** Many scientific systems are not static point clouds. Standard persistent homology often freezes time or scale. This work creates a statistical/ML-compatible summary for time-varying point clouds and multivariate time series.
- **Evidence:** The paper is primarily methodological with illustrative applications; it is not yet a large-scale ML benchmark paper.
- **Limitations:** Computational scalability for very large scientific meshes and usefulness as an end-to-end learned representation are not yet established.
- **Mesh / simulation connection:** Potentially important for moving meshes, adaptive remeshing, fluid interfaces, particle systems, and evolving geometric domains. A future learned AMR policy could distinguish transient remeshing artifacts from persistent changes in solution/domain structure.

### Central Limit Theorems for Persistent Betti Numbers — Tada
- **Primary source:** https://arxiv.org/abs/2609.06510
- **Date:** 2026-09-06.
- **Status:** **New theoretical foundation; indirect ML impact.**
- **Contribution:** Develops a homological-algebraic route to central limit theorems for persistent Betti numbers of random chain complexes, including Poisson/Gibbs point-process settings and several Vietoris–Rips-related invariants.
- **Why it matters:** Topological summaries are increasingly used as statistical observables in ML and scientific computing. Better asymptotic theory clarifies when persistent quantities admit calibrated uncertainty rather than being treated as deterministic feature vectors.
- **Limitations:** No neural architecture and no direct large-scale application. Translation from asymptotic stochastic topology to practical confidence estimates in learned systems remains open.
- **Mesh / simulation connection:** Relevant for stochastic geometry, uncertainty quantification, random point clouds, and any pipeline where topological features of sampled physical states are used as monitoring statistics.

---

## 3. Topology can constrain which neural representations are possible

### Topology Obstructs Pure Foundation Neural Quantum States — Heightman et al.
- **Primary source:** https://arxiv.org/abs/2609.07591
- **Date:** 2026-09-07.
- **Status:** **New, high conceptual importance; narrow empirical domain.**
- **Core result:** For gapped Hamiltonian families with a non-trivial ground-state bundle, any continuous normalized state-vector model must fail somewhere: the paper proves zero fidelity at some parameter value and at least one spectral-gap energy error there. Operator-valued models avoid the obstruction.
- **Why it matters beyond quantum ML:** This is an unusually clean demonstration that a representation class can be **topologically incapable** of representing the target family globally. More width, depth, data, or optimization cannot repair a structural mismatch in the representation.
- **Evidence:** The paper gives theory and demonstrations on one- and two-qubit systems.
- **Limitations:** The direct experiments are tiny quantum systems; generalization to mainstream deep learning is conceptual, not empirically demonstrated.
- **Mesh / geometry connection:** The transferable question is important: does the chosen latent/output parameterization admit every topology or geometric class needed by the task without discontinuities/singular charts? This matters for neural CAD, mesh generation, shape generation, and topology-changing simulation.

### Assessment update
This paper strengthens a broader thesis: topology is not merely useful information to inject into a network; it can determine whether a chosen **representation family is expressive enough in principle**. For geometry generation, representation design should be treated as a topological question before optimizing the network architecture.

---

## 4. Higher-order structure is not reducible to the graph skeleton

### Hidden higher-order vulnerabilities in simplicial complexes revealed by fixed-index spectral robustness — Luo
- **Primary source:** https://journals.aps.org/pre/abstract/10.1103/n4ps-wmd7
- **Version of record:** 2026-09-08, Physical Review E.
- **Status:** **Strengthened this week by peer-reviewed publication; method itself predates this week.** The preprint appeared in March 2026.
- **Construction:** Re-examines robustness based on the Hodge 1-Laplacian. The paper shows that tracking the instantaneous smallest positive eigenvalue can silently switch spectral branches when the kernel changes, and proposes a fixed-index robustness measure plus triangle sensitivity from perturbation theory.
- **Evidence:** Synthetic and empirical clique complexes show that deleting a small fraction of triangles can collapse the monitored higher-order mode while leaving the 1-skeleton and graph-level robustness observables unchanged.
- **Why it matters:** This is strong evidence for a key TDL claim: a graph representation can preserve all node-edge connectivity while missing failures encoded entirely in higher-order cells.
- **Limitations:** It is a robustness/spectral analysis paper, not a learned architecture. The connection to predictive performance of simplicial neural networks is still indirect.
- **Mesh / simulation connection:** A triangular/tetrahedral mesh carries face/cell structure not recoverable from treating it merely as an untyped graph. This supports using simplicial/cellular operators or explicit incidence information for mesh learning and mesh-quality monitoring.

### Assessment update
The common shortcut “mesh = graph” should be considered **deprioritized for topology-sensitive tasks**. Graph encoders can remain useful for local geometric processing, but they should not be assumed to preserve higher-order functional structure when faces/cells matter.

---

## 5. Statistical inference on persistence summaries

### Statistical Inference for Persistence Diagrams via Landmark Embeddings: Minimax Theory and Finite Approximation — Bagchi, Majhi, Mitra, Virk
- **Primary source:** https://arxiv.org/abs/2609.07691
- **Date:** 2026-09-07.
- **Status:** **New; methodological rather than architectural.**
- **Contribution:** Develops covariance estimation, two-sample tests, confidence balls, power guarantees, and finite-dimensional approximation theory for populations of persistence diagrams embedded with PLACE/PALACE-style landmark representations.
- **Why it matters:** Topological ML often reports diagram distances or average topological features without a calibrated inferential layer. This work helps move TDA from descriptive summaries toward population-level statistical claims.
- **Evidence:** Theory, simulations, and an application to resting-state connectivity from ABIDE.
- **Limitations:** It does not directly improve neural predictive performance; practical value depends on whether the chosen persistence embedding captures task-relevant distinctions.
- **Mesh / simulation connection:** Useful for comparing distributions of generated vs reference meshes, checking whether a topology-aware generator preserves population-level structure, and evaluating ensembles of stochastic simulations.

---

## 6. Current architecture anchors (still relevant)

These are not new this week, but remain the best reference points for interpreting new work.

### Periodic Topological Deep Learning for Polymer Design and Discovery — Yadav et al. (2026)
- **Primary source:** https://arxiv.org/abs/2605.26833
- **Status:** **Still relevant / strong scientific application.**
- Uses periodic Vietoris–Rips complexes plus hierarchical simplicial message passing to capture periodicity and many-body interactions absent from ordinary molecular graphs.
- Reports state-of-the-art prediction across multiple polymer properties and, unusually for this area, includes experimental confirmation of predicted chemical trends on novel polymer pairs.
- Conceptual importance: one of the stronger demonstrations that higher-order topological structure can be part of the model representation rather than an auxiliary post-processing feature.

### COSMOS: Continuous Simplicial Neural Networks — Einizade et al. (2025)
- **Primary source:** https://arxiv.org/abs/2503.12919
- **Status:** **Still relevant / architectural anchor.**
- Derives continuous-time/PDE-style dynamics on simplicial complexes and studies stability and oversmoothing.
- Applications include ocean trajectory prediction and deformable-shape regression, making it especially relevant to scientific geometry.

### Topological Autoencoders / Topological Autoencoders++
- **Primary sources:** https://arxiv.org/abs/1906.00722 and https://arxiv.org/abs/2502.20215
- **Status:** **Foundational but partially superseded in interpretation.**
- Topological Autoencoders established differentiable topology-preserving latent regularization.
- Topological Autoencoders++ showed that naïve extension of the original loss to higher-dimensional homology does not automatically preserve cycles, and introduced a PH1-aware correction.
- Current lesson: “topological loss” is not a single generic primitive; what is preserved depends strongly on the homology dimension, filtration, and geometric realization.

---

## 7. Conceptual map for mesh generation and scientific geometry

The strongest synthesis currently is not “replace meshing with persistent homology.” It is to factor the problem into layers with different mathematical responsibilities:

1. **Topological / combinatorial state** — incidence structure, cells, boundary components, homology/cohomology, Hodge operators.
2. **Geometric residual** — coordinates, metric, curvature, anisotropy, sharp features, local feature size.
3. **Physical state** — PDE variables, boundary conditions, error indicators, solver residuals.
4. **Learned policy / generator** — proposes refinement, collapse, movement, decomposition, or latent geometric transformations.
5. **Validity mechanism** — diffeomorphic action space, simplicial/cellular constraints, planar-graph reconstruction, link-condition checks, or classical meshing kernels.
6. **Evaluation** — solver error/cost plus geometric quality plus topological diagnostics, with statistical calibration where possible.

### Current research hypothesis
The most promising form of “topological deep learning for meshing” is likely **not** a model that receives only persistent-homology features. It is a model whose representation and action space preserve the discrete topological structure that meshing algorithms operate on, while topology-aware statistics monitor whether simplification/adaptation destroys task-relevant structure.

### High-value open questions
- Can a simplicial/cellular encoder outperform graph encoders for solver-grade mesh decisions when the 1-skeleton is held fixed?
- Can a generative mesher operate on a library of topology-valid local moves rather than directly predicting arbitrary connectivity?
- Can spatiotemporal persistence distinguish genuine physical topology changes from transient artifacts introduced by adaptive remeshing?
- What is the smallest geometric residual needed once combinatorial/topological information is explicitly represented?
- Can persistent/Hodge quantities serve as useful RL critics without becoming the computational bottleneck?
- Which topology-preserving parameterizations are too restrictive because the true task requires controlled topology changes?

---

## 8. Evidence quality / priority ranking

**High priority**
- Topology Obstructs Pure Foundation Neural Quantum States — strong theorem about representation-class limitations; narrow domain but deep conceptual consequence.
- Spatiotemporal Persistence Landscapes — rigorous new invariant for dynamic data with direct scientific-simulation relevance.
- Periodic-TDL — strong application evidence, including limited experimental validation.

**Medium-high priority**
- IGG geodesic diffusion — useful proof of the “valid transformation space” idea; restricted to fixed topology and partly an extension of prior work.
- Hodge fixed-index robustness — peer-reviewed evidence that higher-order failures can be invisible at graph level.
- Topologically consistent parcel vectorization — strong generative-geometry pattern, but domain-specific and not algebraic TDA.

**Medium / infrastructure**
- Statistical inference for persistence diagrams — important for rigorous evaluation, not a modeling breakthrough.
- Central limit theorems for persistent Betti numbers — foundational probability theory with indirect short-term ML impact.

---

## 9. Compact archive / deprioritized interpretations

- **“Persistent homology features alone are sufficient to make a model topological.” — Deprioritized.** They can help, but current work increasingly shows value in architectural structure, hard validity, and task-specific topology.
- **“A graph encoder is an adequate generic representation for a mesh.” — Weakened.** Higher-order spectral evidence shows that face/cell perturbations can cause functional collapse with an unchanged 1-skeleton.
- **“A topology-aware loss automatically preserves arbitrary topology.” — Weakened.** Results such as Topological Autoencoders++ show preservation guarantees are homology- and construction-specific.
- **“Topology preservation is always desirable.” — False in general.** Diffeomorphic generators are powerful when the task should remain within a topological class, but they are structurally incapable of modeling genuine topology changes. Future generative systems need controlled topology-changing operations when the application requires them.
