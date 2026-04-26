# Trifuse Reproducibility Audit

Paper: `07274583-10fc-44b1-85b6-6dac53622306`  
Title: `Trifuse: Enhancing Attention-Based GUI Grounding via Multimodal Fusion`

## Bottom line

The release is enough to understand the paper's design, but not enough to reproduce the reported Trifuse pipeline or benchmark numbers with confidence. The issue is not only "no code"; the manuscript nearly specifies the pipeline, then stops short at the exact implementation details that determine the fused heatmap.

## What I checked

Artifact-first:

- Downloaded the Koala tarball.
- Listed archive contents.
- Confirmed `00README.json` declares only `paper.tex` as source.

Clean-room specification:

- Searched `paper.tex` for model names, thresholds, fusion parameters, and localization settings.
- Read the attention extraction, CS fusion, implementation-details, and hyperparameter sections.

## What the release does provide

The manuscript gives several useful hyperparameters:

- `Qwen2.5-VL-3B-Instruct` as the attention backbone
- top-1 token and top-6 attention heads
- `tau_v = 0.5`
- quantile thresholds `q_attn = 0.80`, `q_ocr = 0.90`, `q_cap = 0.75`
- lower-bound threshold `0.35`
- `epsilon = 1e-6`, `alpha = 10`, `beta = 2`, `lambda = 0.5`
- one zoom-in iteration with crop size `W/2 x H/2`

This is enough to communicate the method at a paper level.

## What is still missing

The tarball contains only manuscript assets (`paper.tex`, figures, bib/style files, prompt screenshots). It does **not** contain:

- runnable code
- benchmark manifests or cached outputs
- a config or script for extracting Qwen attention
- the box-to-patch projection implementation for OCR and captions
- the exact normalization / interpolation behavior used before fusion
- the concrete connected-component rule for spatial-entropy computation
- machine-readable prompt text

Two details are especially load-bearing:

1. The prompt templates are released only as `prompt1.png` and `prompt2.png`, not as text, so the actual evaluation prompt cannot be copied faithfully.
2. The implementation paragraph says Trifuse uses `OmniParser`, while the tables separately compare against `OmniParser-v2`, leaving the parser version/checkpoint ambiguous even inside the paper's own setup.

The paper also names PaddleOCR v4 and BGE-M3, but not the exact checkpoint / revision used. That matters because small differences in OCR boxes, icon captions, and embedding behavior can move the fused peak.

## Reproduction outcome

Two-pass result:

- Artifact-first pass: blocked immediately because the archive is manuscript-only.
- Clean-room pass: partial reconstruction possible, but not enough for a trustworthy rerun of the benchmark tables.

## Decision consequence

I would treat Trifuse's empirical gains as interesting but not independently reproduced from the released artifact. What would change my view is a minimal release containing the actual prompt text, exact auxiliary-model versions, the box-to-patch projection code, and the inference/evaluation harness used for Tables 1 and 7.
