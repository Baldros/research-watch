# Complexity-Guided Neural Architecture Selection — Living Research

_Last reviewed: 2026-09-22_

## Central question

Can measurable, interpretable properties of a task, dataset, or learned representation constrain neural-network architecture — especially **width** and **depth** — before or during training, so that architecture selection becomes a structured inference problem rather than blind search?

The strongest current formulation is **not** “one scalar complexity determines the optimal network.” The evidence increasingly supports a **regime-aware complexity vector** whose components constrain different architectural degrees of freedom:

- intrinsic/effective dimension -> representation-width pressure;
- compositional / algorithmic structure -> depth pressure and width-depth tradeoffs;
- topology / geometry -> expressivity constraints and transformation requirements;
- Fisher / information geometry -> task similarity, operator importance, effective capacity, and search-space restriction;
- symmetry / group-orbit structure -> an alternative regime when manifold-based descriptors fail;
- learned task embeddings -> empirical priors over architectures.

A useful long-term target is:

`task/data -> regime identification -> interpretable complexity descriptors -> prior over (width, depth, family) -> targeted training -> capacity diagnostics -> refinement`

No mature unified method currently performs this full mapping in an architecture-family-aware and broadly validated way.

---

# Executive state of the thesis

## Width

**Current status: supported, but regime-dependent.**

The strongest direct evidence remains that **intrinsic dimension can control sufficient width** in restricted settings. AISTATS 2026 gives a theorem where shallow nonlinear width scales polynomially with intrinsic rather than ambient dimension.

However, a new September 2026 result on finite symmetry orbits shows that nearest-neighbor intrinsic dimension can become undefined or resolution-dependent and that hidden-width scaling can follow a different law. This materially weakens any universal claim of the form:

`intrinsic dimension -> width`

and replaces it with:

`regime identification -> appropriate structural descriptor -> width prior`.

## Depth

**Current status: increasingly distinct from width, but harder to estimate from raw data.**

Depth-hierarchy results, compositional approximation theory, and recent work on recursive/loop depth all support the idea that depth expresses a qualitatively different resource from width. The best candidates for a depth-oriented complexity variable are target compositionality, algorithmic structure, transformation hierarchy, and geometric/non-smooth structure of the target.

What is still missing is a robust finite-data estimator that says, before fitting a large model, “this task plausibly requires depth in this range.”

## Topology / geometry

**Current status: useful for constraints and diagnostics, not sufficient alone.**

The literature supports topology as a language for:
- lower/upper expressivity constraints;
- tracking representation simplification across layers;
- measuring structural properties of the architecture graph itself;
- allocating capacity non-uniformly across layers.

But topology is not consistently monotonic with model size, and static input topology is too coarse to serve as a universal architecture selector.

## Fisher / information geometry

**Current status: materially strengthened.**

Fisher is no longer just a speculative post-training diagnostic. There is now direct prior art for:
- Fisher-derived **task distance** reducing NAS search space;
- Fisher-guided transfer of architectures;
- Fisher-based zero-shot architecture scoring;
- Fisher-guided operation selection;
- Fisher-Rao geodesic pruning / capacity removal.

The key limitation remains: most Fisher quantities are **probe/model dependent**. Existing work does not yet derive a general explicit mapping from task Fisher information to width/depth.

---

# Evidence map

## A. Intrinsic dimension -> width

### 1. Linearly Separable Features in Shallow Nonlinear Networks: Width Scales Polynomially with Intrinsic Data Dimension
Alec S. Xu, Can Yaras, Peng Wang, Qing Qu. **AISTATS 2026**.

Primary source: https://proceedings.mlr.press/v300/xu26a.html

The paper models inputs as a union of low-dimensional subspaces and proves that one random nonlinear layer with quadratic activation can make the classes linearly separable with high probability when network width scales polynomially with the **intrinsic dimension** rather than ambient dimension.

**Connection to thesis:** unusually close to the desired form `measurable property of data -> sufficient width`.

**Evidence maturity:** peer-reviewed theorem + experiments.

**Limitations:** shallow network, quadratic activation, union-of-subspaces model, random weights, sufficient-width rather than optimal-width result.

**Status:** **STRENGTHENED / central evidence for the width axis.**

### 2. Width-Independent Compressibility of Deep Neural Networks
Hong-Yi Wang, Mingze Wang, Liu Ziyin. arXiv, 2026-08-22.

Primary source: https://arxiv.org/abs/2608.21752

For deep MLPs with analytic activations, a wide trained network can be approximated at the same depth by a narrower network whose required width depends on an effective input dimension rather than the teacher's original width.

**Connection to thesis:** supports the view that usable width can be governed by effective input structure rather than raw overparameterization.

**Evidence maturity:** theoretical preprint.

**Limitations:** post-hoc compression / existence setting; starts with an already trained teacher; restrictive activation assumptions.

**Status:** **STILL RELEVANT.**

### 3. NEW — Symmetry without a manifold: intrinsic dimension on orbits
Chon-Fai Kam, Miloud Bessafi, Frédéric Cadet. arXiv, 2026-09-15.

Primary source: https://arxiv.org/abs/2609.17926

On modular addition over finite group orbits, the nearest-neighbor intrinsic-dimension estimator can be undefined because symmetry makes nearest-neighbor distance ratios degenerate. Once symmetry is perturbed, the estimate can track resolution rather than a scale-free dimension. In the experiments, hidden-width scaling is better described by an exponential saturation law than by a manifold-style power law.

**Connection to thesis:** this is a direct caution against treating intrinsic dimension as a universal width descriptor. Some tasks are structured primarily by **symmetry / algebraic orbit structure**, not by a generic smooth manifold.

**Evidence maturity:** fresh preprint; narrow but mathematically explicit setting.

**Limitations:** modular arithmetic / finite orbit tasks; does not invalidate intrinsic-dimension results on manifold-like data.

**Status:** **WEAKENS universality of intrinsic dimension; STRENGTHENS a regime-aware complexity-vector view.**

---

## B. Compositional / algorithmic structure -> depth and width-depth tradeoffs

### 4. A Depth Hierarchy for Computing the Maximum in ReLU Networks via Extremal Graph Theory
Itay Safran. **COLT 2026**.

Primary source: https://proceedings.mlr.press/v336/safran26a.html

For exact computation of the maximum over (d) inputs, the paper proves a depth hierarchy: for depth (k), a super-linear width lower bound is necessary over a range of depths. The proof ties architectural requirements to the geometric/combinatorial structure of the target function's non-differentiable ridges.

**Connection to thesis:** rigorous example where **target structure induces a quantitative width-depth tradeoff**. Width and depth are not interchangeable parameter counts.

**Evidence maturity:** peer-reviewed theorem.

**Limitations:** exact representation of one special target; not a finite-sample task estimator.

**Status:** **STRENGTHENED / strong evidence for a separate depth axis.**

### 5. Optimal Neural Network Approximation of Smooth Compositional Functions on Sets with Low Intrinsic Dimension
Thomas Nagler, Sophie Langer. **COLT 2026**.

Primary source: https://proceedings.mlr.press/v336/nagler26a.html

The paper combines low Minkowski/intrinsic dimension of the domain with **compositional structure** of the target. Approximation and statistical rates depend jointly on smoothness, intrinsic dimension, and the hardest component in the composition.

**Connection to thesis:** supports a **multi-component complexity model**: data geometry and target compositionality are separate sources of difficulty.

**Evidence maturity:** peer-reviewed theory.

**Limitations:** does not directly infer an optimal `(width, depth)` pair from observed data; target structure must be known or estimated.

**Status:** **STRENGTHENED.**

### 6. Algorithmic Foundations of Deep Learning: Complexity-Theoretic Rates and a Characterization of Universal Approximation
Anastasis Kratsios, Simone Brugiapaglia, Bum Jun Kim, Gregory Cousins, Haitz Saez de Ocariz Borde. arXiv, 2026-06-25.

Primary source: https://arxiv.org/abs/2606.26705

Treats networks as computational devices and relates network depth, width, and parameter count to the depth, width, gate count, and structure of real-valued circuits computing the target.

**Connection to thesis:** possible formal basis for a `compositional / algorithmic complexity -> architecture` axis.

**Evidence maturity:** theoretical preprint.

**Limitations:** translating circuit complexity into a reliable estimator from finite data remains unresolved.

**Status:** **STILL RELEVANT.**

### 7. Compositionality Emerges in a Narrow Depth-Connectivity Regime: Architecture Constraints and Solution Manifolds
Dat H. Do, Rushi Shah, Duc V. Le, Dianbo Liu. arXiv, 2026-06-18.

Primary source: https://arxiv.org/abs/2606.19941

The paper reports that compositional solutions emerge only in a narrow, target-dependent regime of depth and connectivity, with shallower and deeper networks sometimes converging to structurally different non-compositional solutions. It also proposes a heuristic depth predictor and similarity-based pruning.

**Connection to thesis:** directly supports the idea that there may be **task-dependent depth regimes**, not merely monotonic gains from more depth.

**Evidence maturity:** preprint; empirical + theoretical arguments.

**Limitations:** target family and compositional benchmark design matter heavily; heuristic predictor is not yet a general task-complexity estimator.

**Status:** **PROMISING, but lower-confidence than the COLT theory.**

---

## C. Geometry / topology -> layerwise capacity allocation and architectural structure

### 8. Geometry-Guided Layerwise FFN Width Allocation in Transformers
Timur Mudarisov, Mikhail Burtsev, Radu State. arXiv, 2026-08-03.

Primary source: https://arxiv.org/abs/2608.02064

The paper measures how much each Transformer FFN changes the geometry of token representations using correspondence-preserving shift, Gromov-Wasserstein distortion, and persistent-homology quantities. These measurements are used to **allocate FFN width non-uniformly across layers under a fixed parameter budget**.

Across seven pretrained models, Gromov-Wasserstein work is more consistently associated with perturbation-based layer sensitivity than the finite-sample topological estimate. In paired training runs, several geometry-based width schedules outperform uniform width and hand-designed tapering.

**Connection to thesis:** this is one of the closest recent results to the core idea. It demonstrates:

`measurable representation geometry -> layerwise width allocation`

rather than merely “geometry correlates with performance.”

**Evidence maturity:** preprint with multiple model sizes and paired training runs.

**Limitations:** measurements come from a forward pass / existing model behavior, so this is not purely data-first before instantiating a model; it reallocates a fixed width budget rather than predicting total width.

**Status:** **NEW / HIGH RELEVANCE. Strengthens geometry as an actionable architecture signal.**

### 9. NEW — From complexity to simplicity: Topological insights for neural architecture search
Yan Dai et al. **Applied Soft Computing**, available online 2026-09-07; journal issue 2027.

Primary source: https://doi.org/10.1016/j.asoc.2026.116389

Architectures are represented as DAGs. Spectral/Jacobian analysis motivates two simple structural indices:
- **Connectivity Depth**
- **Connectivity Breadth**

The indices are data-free and training-free and are used to screen NAS search spaces before expensive evaluation. Reported experiments reduce search cost while preserving or improving accuracy.

**Connection to thesis:** important because it formalizes **interpretable depth/breadth structure** and uses it to constrain architecture search.

**But:** the measured complexity is a property of the **candidate architecture**, not of the task/data. It therefore addresses the complementary direction:

`architecture structure -> trainability / search-space filtering`

rather than the central desired map:

`task/data complexity -> architecture constraints`.

**Evidence maturity:** peer-reviewed journal article / in-press final publication path.

**Limitations:** structural screening still sits inside NAS; does not explain which task requires which depth/breadth.

**Status:** **NEW / RELEVANT COMPLEMENT, not direct validation of data-first selection.**

### 10. Low-dimensional topology of deep neural networks
Junyu Ren, Lek-Heng Lim. 2026.

Primary source: https://arxiv.org/abs/2606.31856

Fixes representation width at dimension three to isolate depth and activation effects, then studies how different network families alter linking numbers.

**Connection to thesis:** methodologically valuable because it deliberately separates width from depth and gives a topological language for depth-related expressivity.

**Evidence maturity:** 2026 research result.

**Status:** **STILL HIGHLY RELEVANT.**

### 11. Topological Expressivity of ReLU Neural Networks
Ekin Ergen, Moritz Grillo. **COLT 2024**.

Primary source: https://proceedings.mlr.press/v247/ergen24a.html

Provides upper/lower bounds on topological simplification and depth advantages for certain topological transformations.

**Status:** **FOUNDATIONAL / still relevant.**

### 12. On Characterizing the Capacity of Neural Networks using Algebraic Topology
William H. Guss, Ruslan Salakhutdinov. arXiv, 2018.

Primary source: https://arxiv.org/abs/1802.04443

One of the earliest explicit **data-first architecture selection** proposals: use persistent homology / topological complexity of labeled data and relate it empirically to architecture capacity.

**Status:** **FOUNDATIONAL but incomplete.** Later topology work has strengthened expressivity theory, but a mature general selector did not emerge from topology alone.

---

## D. Fisher information / effective dimension / information geometry

### 13. Fisher Task Distance and Its Application in Neural Architecture Search
Cat P. Le, Mohammadreza Soltani, Juncheng Dong, Vahid Tarokh. arXiv, 2021.

Primary source: https://arxiv.org/abs/2103.12827

Defines an asymmetric task distance from Fisher information matrices and uses it to identify related previous tasks. Their optimized architectures are then reused to reduce the target-task NAS search space.

**Connection to thesis:** this is crucial prior art. It already realizes:

`Fisher-derived task geometry -> architecture prior / reduced search space`.

It does **not** derive width/depth from Fisher information directly, but it substantially narrows the novelty space around “Fisher can guide architecture selection.”

**Evidence maturity:** established research result with theory + experiments.

**Status:** **FOUNDATIONAL; should be treated as central prior art for the Fisher branch.**

### 14. Fisher duty interval and particle swarm optimization for neural architecture search
Ali Delshadi, Vahid Mehrdad, Mohammad Bagher Dowlatshahi. **Scientific Reports**, 2026-05-19.

Primary source: https://www.nature.com/articles/s41598-026-53204-0

Uses Fisher-information-based task distance to find previously solved tasks, transfer successful architectures, initialize/narrow a particle-swarm NAS search, and reduce candidate-evaluation cost.

**Connection to thesis:** recent peer-reviewed evidence that Fisher-derived task similarity can practically constrain architecture search.

**Evidence maturity:** peer-reviewed journal article.

**Limitations:** Fisher representation is still probe/model dependent; PSO remains a search stage; architecture is transferred from similar tasks rather than derived from first principles.

**Status:** **STRENGTHENS Fisher as a practical architecture prior.**

### 15. Fisher-DARTS: A Neural Architecture Search Framework with Fisher Information Optimization
Yu Zhang, Changyuan Wang. **Applied Sciences**, 2026-04-14.

Primary source: https://doi.org/10.3390/app16083808

Uses empirical/diagonal Fisher information to rank candidate operations and guide differentiable NAS.

**Connection to thesis:** Fisher can influence architectural choices directly during search.

**Limitations:** this is Fisher as an internal NAS optimization signal, not task-complexity measurement.

**Status:** **RELEVANT, but less directly data-first.**

### 16. FaB: Function-aware Fisher information proxy for training-free neural architecture search
Chan Sik Han, Sun Woo Jeong, Keon Myung Lee. SSRN preprint, posted 2026-06-04.

Primary source: https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6880907

The method argues that many zero-cost proxies implicitly compare gradients in architecture-specific Euclidean parameter spaces. FaB instead defines a Fisher geometry on predictive distributions, aiming to compare architectures in a function-space metric shared across candidates.

**Connection to thesis:** conceptually important because it addresses a central weakness of Fisher-based architecture comparison: **parameterization dependence**.

**Evidence maturity:** working paper / SSRN, not peer-reviewed.

**Limitations:** training-free proxy evaluation rather than task-to-width/depth inference.

**Status:** **PROMISING / low-to-moderate maturity.**

### 17. NEW — Optimal Pruning for Neural Architectures using Fisher Information Distances
David S. Berman, Yen-Yu Fu, Edward Hirst, Thelma Chiwete Obirai. arXiv, 2026-09-14.

Primary source: https://arxiv.org/abs/2609.16129

Models pruning as movement to a parameter-zero hypersurface and measures the displacement using geodesic distance induced by the Fisher metric. More faithful geodesic approximations outperform magnitude pruning and local-Fisher-only pruning on tested fully connected networks and vision transformers.

**Connection to thesis:** strong support for using Fisher-Rao geometry to quantify **how much architectural capacity can be removed without materially changing model behavior**.

This fits naturally as:

`candidate architecture -> Fisher geometry -> removable/redundant capacity -> architecture refinement`.

**Evidence maturity:** fresh preprint; multiple architecture/dataset combinations, but modest benchmark scale.

**Limitations:** starts from a trained model; it is not a pre-training task-to-width/depth rule.

**Status:** **NEW / STRENGTHENS Fisher as a post-training capacity diagnostic.**

### 18. Effective dimension of machine learning models
Amira Abbas, David Sutter, Alessio Figalli, Stefan Woerner. 2021.

Primary source: https://arxiv.org/abs/2112.04807

Defines local effective dimension from Fisher information and relates it to generalization/capacity.

**Connection to thesis:** provides mathematical machinery for quantifying effective degrees of freedom.

**Limitations:** capacity/generalization measure rather than direct architecture prescription.

**Status:** **STILL RELEVANT as mathematical infrastructure; direct width/depth mapping remains unproven.**

---

## E. Dataset-to-architecture / task embeddings / meta-NAS

### 19. Task2Vec: Task Embedding for Meta-Learning
Achille et al. ICCV 2019.

Primary source: https://openaccess.thecvf.com/content_ICCV_2019/html/Achille_Task2Vec_Task_Embedding_for_Meta-Learning_ICCV_2019_paper.html

Uses a Fisher-derived embedding of tasks.

**Connection to thesis:** foundational evidence that Fisher information can encode task relationships in a reusable embedding space.

**Status:** **FOUNDATIONAL.**

### 20. MetaD2A: Rapid Neural Architecture Search by Learning to Generate Graphs from Datasets
Hayeon Lee, Eunyoung Hyung, Sung Ju Hwang. ICLR 2021.

Primary source: https://arxiv.org/abs/2107.00860

Learns a direct dataset-to-architecture latent mapping and a meta-performance predictor.

**Connection to thesis:** proves that `dataset -> architecture` is empirically learnable.

**Interpretability gap:** the dataset embedding is learned rather than decomposed into explicit mathematical complexity coordinates.

**Status:** **FOUNDATIONAL empirical baseline.**

### 21. Dataset2Vec
Primary source: https://link.springer.com/article/10.1007/s10618-021-00737-9

Learns dataset embeddings for meta-learning / hyperparameter optimization.

**Status:** **STILL RELEVANT as a black-box baseline.**

---

# Material changes in the 2026-09-22 review

## 1. The width hypothesis became more conditional
The 2026-09-15 **Symmetry without a manifold** result is a meaningful correction to the simple narrative that intrinsic dimension is the natural width variable. It shows a concrete regime where standard intrinsic-dimension estimation is structurally inappropriate.

**Revision:** replace “intrinsic dimension is the width variable” with:

`identify structural regime first; use intrinsic dimension only in manifold-like regimes`.

Symmetry/group/orbit descriptors should be added to the candidate complexity vector.

## 2. Geometry gained a direct architecture-allocation result
**Geometry-Guided Layerwise FFN Width Allocation** is highly relevant because it does not merely correlate geometry with model quality; it uses representation geometry to redistribute actual FFN width across Transformer depth.

**Revision:** geometry should move from “diagnostic” to **actionable architecture signal**, albeit currently model-mediated rather than pure data-first.

## 3. Fisher is now clearly architecture-relevant, but not the missing scalar
The combination of:
- Fisher Task Distance (2021),
- Scientific Reports Fisher-guided NAS (2026),
- Fisher-DARTS (2026),
- function-aware Fisher zero-shot NAS,
- and Fisher-Rao pruning (2026-09-14)

shows that Fisher information already spans **task similarity, search-space restriction, candidate scoring, operator choice, and capacity refinement**.

**Revision:** the open problem is no longer “can Fisher guide architecture?” It can. The harder question is:

> Can a Fisher-derived, sufficiently model-invariant descriptor predict explicit width/depth ranges rather than merely rank, retrieve, or prune candidate architectures?

## 4. Architecture-side complexity also matters
The 2026-09-07 Applied Soft Computing result introduces interpretable **Connectivity Depth** and **Connectivity Breadth** indices and uses them to screen NAS spaces.

**Revision:** a full framework likely needs both:
- **task-side descriptors**: what complexity the task demands;
- **architecture-side descriptors**: what structural capacity/trainability the candidate provides.

Then architecture selection becomes a **matching problem** rather than a one-sided prediction problem.

---

# Updated working hypothesis

The most plausible research program is now:

1. **Identify task/data regime**
   - manifold-like geometry;
   - symmetry/group-orbit structure;
   - compositional / algorithmic structure;
   - class-boundary topology;
   - mixed regimes.

2. **Measure a task-side complexity vector**
   - intrinsic/effective dimension when valid;
   - symmetry/orbit descriptors;
   - class-conditional or decision-boundary topology;
   - local geometry / curvature / Gromov-Wasserstein work;
   - compositional proxies;
   - conditional uncertainty/noise;
   - Fisher-derived task distance or task embedding.

3. **Measure an architecture-side capacity vector**
   - width / layerwise width profile;
   - static depth and usable/recursive depth;
   - Connectivity Depth / Connectivity Breadth;
   - topological expressivity descriptors;
   - Fisher effective capacity / removable directions.

4. **Learn or derive a compatibility map**
   - not a single “optimal architecture”;
   - preferably a prior or feasible region over architecture families and `(width, depth)`.

5. **Refine after limited training**
   - Fisher-Rao pruning;
   - representation-geometry diagnostics;
   - topology of layerwise simplification;
   - effective-dimension diagnostics.

This is more defensible than a direct scalar map:

`C(D) -> (W, L)`.

A better target is:

`C_task(D) + C_arch(A) -> compatibility(D, A)`.

---

# Key unresolved questions

1. Can a pre-training diagnostic determine whether manifold geometry, symmetry, compositionality, topology, or another regime dominates the task?
2. Which task properties predict **width independently of depth**, and vice versa?
3. Can compositional/algorithmic complexity be estimated reliably from finite supervised data without first fitting a large model?
4. Can Fisher task geometry be made less dependent on probe architecture?
5. Does Fisher-derived task distance predict explicit width/depth, or mainly retrieve good architectures from similar solved tasks?
6. Should topology be measured on raw inputs, class-conditional sets, estimated decision boundaries, or representation trajectories?
7. Can representation geometry measured on a small probe model transfer to a larger architecture and guide width allocation reliably?
8. How should residual structure, attention, activation choice, recurrence, and looped depth enter the architecture-capacity vector?
9. Can interpretable complexity descriptors match or complement MetaD2A / Dataset2Vec-style learned task embeddings while reducing search cost?
10. What should count as success: prediction of exact architecture, a narrow feasible region, lower/upper bounds, search-cost reduction, or stable transfer across datasets?

---

# Deprioritized / weakened claims

- **“One scalar task complexity determines the optimal architecture.”**  
  **DEPRIORITIZED.** Evidence favors multiple interacting descriptors.

- **“Intrinsic dimension alone predicts width.”**  
  **WEAKENED.** Strong in some geometric regimes, invalid or misleading in symmetry-dominated regimes.

- **“Topology alone is enough.”**  
  **WEAKENED.** Topology gives useful constraints and diagnostics but is not consistently monotonic with width/depth.

- **“Fisher is only useful after training.”**  
  **SUPERSEDED.** Fisher is already used for task distance, search-space restriction, zero-shot scoring, operation selection, and pruning.

- **“Fisher directly tells us the correct width/depth.”**  
  **STILL UNSUPPORTED.** Existing methods rank, retrieve, prune, or restrict search rather than produce a general explicit width/depth prescription.

- **“Width and depth are interchangeable parameter budget.”**  
  **STRONGLY WEAKENED.** Depth hierarchies, intrinsic-dimension width results, compositional approximation theory, and architecture-structure results all support qualitatively distinct roles.

---

# Compact historical anchors

These remain useful and should not be removed even when newer work supersedes parts of them:

- Guss & Salakhutdinov (2018), algebraic topology for neural capacity / data-first architecture selection: https://arxiv.org/abs/1802.04443
- Task2Vec (2019), Fisher-derived task embeddings: https://openaccess.thecvf.com/content_ICCV_2019/html/Achille_Task2Vec_Task_Embedding_for_Meta-Learning_ICCV_2019_paper.html
- Fisher Task Distance + NAS (2021): https://arxiv.org/abs/2103.12827
- MetaD2A (2021), learned dataset-to-architecture mapping: https://arxiv.org/abs/2107.00860
- Effective dimension from Fisher information (2021): https://arxiv.org/abs/2112.04807
- Ergen & Grillo (2024), topological expressivity of ReLU networks: https://proceedings.mlr.press/v247/ergen24a.html
