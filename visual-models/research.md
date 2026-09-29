# Visual Models Research Watch

_Last reviewed: 29 September 2026_

_Current ecosystem coverage: Meta FAIR / Meta AI. Additional research ecosystems will be incorporated into this same document over time._

## Current picture

Meta's vision research currently looks less like one dominant model family and more like several interacting lines. The clearest representation-learning lineage is:

DINO/DINOv2/DINOv3 -> Perception Encoder -> EUPE -> MoE-ViE

The design trend is moving from strong specialist representations toward consolidation and efficient scaling: DINO-style dense spatial features, PE-style semantic/language-aligned visual features, EUPE-style multi-teacher distillation, and now sparse MoE capacity in the vision encoder itself.

### Current assessment

| Claim | Status | Confidence | Best evidence |
| --- | --- | ---: | --- |
| Meta is consolidating specialist visual representations into reusable universal encoders | Strengthened | High | EUPE |
| Sparse conditional capacity is now a serious Meta vision-encoder scaling direction | New | Moderate-high | MoE-ViE |
| Recent universal-encoder gains cannot be attributed to architecture alone | Strongly supported | High | EUPE data/teacher pipeline; MoE-ViE kernels |
| Generalist VLMs are challenging task-specific 3D designs, but have not displaced them | New, bounded | Moderate | VLM3 |
| Streaming egocentric memory/temporal grounding remains a major bottleneck | Strengthened | Moderate-high | S-EMBER |
| Frequency-aware generation is a promising but early Meta generative-vision direction | New, preliminary | Moderate | WaiT |

## High-priority current work

### MoE-ViE: Mixture of Experts Vision Encoder for Efficient Image and Video Understanding

- **Authors:** Bonan Zhang, Shiyu Dong, Quan Hung Tran, Katharina Gschwind, Shuqi Yang, Sijia Chen, Adel Ahmadyan, Seungwhan Moon, Lu Zhang, Ahmed Kirmani, Babak Damavandi, Anuj Kumar
- **Date/status:** 18 August 2026; arXiv; official repository describes it as an ECCV 2026 paper
- **Primary sources:** https://arxiv.org/abs/2608.17402 and https://github.com/facebookresearch/moe_vie
- **Artifacts:** official code, model definitions, checkpoints/model cards, zero-shot evaluation suite
- **Status in this map:** New, high priority

MoE-ViE systematically studies sparse Mixture-of-Experts scaling for CLIP-style image/video encoders. The technical package combines fine-grained experts, auxiliary-loss-free expert balancing, custom Triton kernels, and frame-level distillation/freezing for video.

The important novelty is not just having more sparse parameters. Meta is trying to make conditional capacity useful in wall-clock inference by co-designing the encoder and kernels. The paper reports that all released MoE sizes outperform their dense counterparts; the largest matches a state-of-the-art encoder 1.7x its size at 76% of its reported latency and improves image/video VLM results even against encoders with substantially more activated parameters.

**Interpretation:** this opens a new sparse-scaling branch next to Meta's dense DINO/PE-style encoder lineage.

**Limitations:** latency is hardware/kernel dependent; fair comparison must separate total parameters, activated parameters, memory traffic, resolution, batch size, and custom-kernel maturity. The public evidence does not yet show the same efficiency advantage on mobile/edge or non-H100-class hardware.

### Efficient Universal Perception Encoder (EUPE)

- **Authors:** Chenchen Zhu, Saksham Suri, Cijo Jose, Maxime Oquab, Marc Szafraniec, Wei Wen, Yunyang Xiong, Patrick Labatut, Piotr Bojanowski, Raghuraman Krishnamoorthi, Vikas Chandra
- **Date/status:** 23 March 2026; arXiv
- **Primary sources:** https://arxiv.org/abs/2603.22387 and https://github.com/facebookresearch/EUPE
- **Status in this map:** Strengthens the universal-encoder line

EUPE uses a "scale up, then scale down" distillation strategy. Multiple expert visual foundation models are first consolidated into a large proxy teacher, then distilled into a smaller efficient encoder and adapted across resolutions.

This is mainly **training/distillation novelty**, not a new backbone architecture. Its conceptual importance is that Meta explicitly treats semantic/language-aligned and dense/spatial visual expertise as complementary representations worth merging into one deployable encoder.

**Limitations:** the result depends on very strong teachers and Meta-scale data. Training cost is much larger than inference cost, and outside reproduction is constrained by the data/teacher pipeline. Improvements therefore should not be attributed to student architecture alone.

### VLM3: Vision Language Models Are Native 3D Learners

- **Authors:** Zhipeng Cai, Zhuang Liu, Yunyang Xiong, Zechun Liu, Vikas Chandra, Yangyang Shi
- **Date/status:** 28 May 2026; arXiv; official repository labels it NeurIPS 2026
- **Primary sources:** https://arxiv.org/abs/2605.30561 and https://github.com/facebookresearch/VLM3
- **Status in this map:** New conceptual challenge to specialist 3D modeling

VLM3 argues that standard VLMs can learn depth, pixel correspondence, camera pose, and object-level 3D reasoning without task-specific 3D heads, specialized losses, or heavy architectural modifications. The main ingredients are focal-length unification, normalized text-based pixel references, and data mixture/scaling.

The novelty is intentionally **not architectural**. The claim is that task formulation and data can unlock 3D behavior already representable by a general VLM.

**Limitations:** it uses a non-Meta base VLM; some training uses large internal data; official model weights are not yet generally available in the repository. That makes the strongest claims harder to reproduce and leaves data scale as a major explanatory variable. "Native 3D learner" should not be read as "off-the-shelf VLM needs no targeted 3D supervision."

### S-EMBER: A Large-Scale Benchmark for Streaming Egocentric Memory Retrieval

- **Authors:** Xiaodong Wang, Xuanyi Zhao, Pedro Rodriguez, Devendra Singh Sachan, Barlas Oguz, Seungwhan Moon, Shang-Wen Li, Gargi Ghosh, Xin Dong, Wen-Tau Yih
- **Date/status:** 2 July 2026; revised 26 August 2026; arXiv benchmark paper
- **Primary source:** https://arxiv.org/abs/2607.02689
- **Status in this map:** High-priority evaluation signal for wearable/egocentric systems

S-EMBER contains 3,141 egocentric videos, 388 hours of Ray-Ban Meta smart-glasses footage, and 9,448 QA pairs with temporal evidence localization. It replaces offline full-video retrieval with a causal streaming setting closer to wearable deployment.

The strongest tested models reach less than half the human rate when both the answer and its temporal localization must be correct.

**Interpretation:** merely scaling the visual/VLM backbone does not yet solve streaming episodic memory. Retrieval, memory selection, temporal grounding, and causal state management are likely separate architectural problems.

### WaiT for the Signal: Simple Frequency-Aware Flow-Matching

- **Authors:** Krunoslav Lehman Pavasovic, Theophane Vallaeys, Stephane Mallat, Giulio Biroli, Luke Zettlemoyer, Brian Karrer, Jakob Verbeek
- **Date/status:** 4 August 2026; arXiv
- **Primary source:** https://ai.meta.com/research/publications/wait-for-the-signal-simple-frequency-aware-flow-matching/
- **Status in this map:** New, preliminary generative-vision branch

WaiT uses a wavelet-aware Transformer and flow-matching schedule in which high-frequency bands remain noise until coarse structure has emerged, then join the generation process. Meta reports ImageNet-512 pixel-space FID 1.43, FID 1.3 for its 2B model, up to 50% lower sampling compute in its comparisons, and extension to OpenImages and Kinetics-600 video.

The deeper idea is a frequency-structured allocation of generative compute rather than uniform treatment of all spatial frequencies.

**Limitations:** the work is preprint-stage and introduces a native-resolution multi-axis evaluation partly because conventional FID can hide texture loss. Independent matched-compute reproduction is still important.

## Conceptual synthesis

### 1. Universal perception is becoming a consolidation problem

EUPE suggests Meta no longer treats semantic/language-aligned representation and dense spatial representation as separate endpoints. The goal is to transfer complementary specialist behavior into one efficient encoder.

### 2. Sparse scaling has reached the visual encoder

MoE-ViE is the clearest recent architecture-level change. The next important evidence is whether sparse gains survive equalized compute and diverse hardware, not just whether benchmark scores improve.

### 3. 3D is splitting between specialist and generalist strategies

VLM3 makes a strong generalist claim: targeted data/interface design may remove the need for specialized 3D heads. This does not yet invalidate specialist geometry architectures, especially in high-resolution, low-latency settings. Matched data/compute/latency comparisons are more informative than raw leaderboard positions.

### 4. Wearable vision exposes memory as a first-class perception problem

S-EMBER shows that accurate frame/image representation is insufficient when the system must remember, retrieve, and temporally ground evidence from a continuous causal stream.

## Historical anchors

- **DINOv3** — current anchor for Meta's large-scale self-supervised dense/spatial representation lineage: https://ai.meta.com/research/publications/dinov3/
- **Perception Encoder** — bridge between contrastive visual-language encoders and broadly useful image/video/dense representations: https://arxiv.org/abs/2504.13181
- **Segment Anything lineage** — promptable segmentation/detection/tracking remains a distinct major Meta vision branch: https://github.com/facebookresearch/sam3

## Watch triggers

Future runs should prioritize:
1. independent or cross-hardware MoE-ViE latency evidence;
2. fair dense-vs-MoE comparisons at matched activated compute;
3. EUPE-style reproduction without Meta-scale private/curated data;
4. public VLM3 weights and controlled ablations removing internal data;
5. specialist-vs-generalist 3D comparisons at matched resolution/latency/compute;
6. architectures that materially improve S-EMBER rather than merely scaling the backbone;
7. a post-DINOv3 self-supervised objective or successor visual foundation model;
8. convergence between DINO/PE/EUPE, SAM, 3D, and wearable vision backbones.

## Source index

- Meta AI computer-vision publications: https://ai.meta.com/results/?content_types%5B0%5D=publication&research_areas%5B0%5D=computer-vision
- Meta Research GitHub: https://github.com/facebookresearch
- MoE-ViE: https://arxiv.org/abs/2608.17402
- EUPE: https://arxiv.org/abs/2603.22387
- VLM3: https://arxiv.org/abs/2605.30561
- S-EMBER: https://arxiv.org/abs/2607.02689
- WaiT: https://ai.meta.com/research/publications/wait-for-the-signal-simple-frequency-aware-flow-matching/
