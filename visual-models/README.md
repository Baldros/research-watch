# Visual Models Research Watch

This research stream tracks substantial advances in computer vision and visual representation learning across major research ecosystems. The initial coverage is **Meta FAIR / Meta AI**, with other ecosystems to be added over time without changing the repository structure.

## Current ecosystem coverage

### Meta FAIR / Meta AI

Track Meta's major visual-model lineages and closely related work by Meta researchers, including visual foundation models, self-supervised learning, ViTs and sparse/MoE encoders, detection and segmentation, image/video understanding, visual embeddings, 3D and spatial perception, egocentric/embodied vision, and VLM work when the contribution materially concerns vision.

Current Meta lines of interest include:

1. Universal visual representations: DINO, DINOv2/DINOv3, Perception Encoder, EUPE, and sparse/MoE scaling.
2. Promptable segmentation/detection: Segment Anything and successors.
3. 3D and spatial perception.
4. Video, egocentric, and embodied perception.
5. Vision-language and multimodal modeling when the visual component is technically substantive.
6. Generative vision when it introduces substantive model, training, or evaluation ideas.

Future ecosystems can be added here as additional sections rather than separate directories.

## Source hierarchy

Prioritize peer-reviewed papers and primary preprints, then official research pages and technical reports, then official repositories, weights, datasets, model cards, and benchmarks. Researcher blogs and talks are secondary synthesis or discovery sources.

## Evidence policy

Separate architectural novelty from training/distillation, data/scale, evaluation, and systems/kernel effects. Check claims against model size, activated parameters, data, resolution, compute, adaptation, test-time compute, and hardware when possible. Flag weak ablations, proprietary data, unreleased artifacts, missing compute disclosure, non-peer-reviewed work, and marketing claims beyond the evidence.

## Research memory

`research.md` is the living research map for the whole visual-models program. Each run should revise it in place, preserve primary links and useful history, deduplicate sources, update prior assessments when evidence changes, and make the ecosystem/source of each result explicit.
