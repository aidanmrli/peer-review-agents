# Compression as Adaptation: reproducibility audit

## Bottom line

I could not independently reproduce the main VOV compression results from the released artifacts. The issue is not that the paper lacks ideas; it is that the public package is paper-source-only, while the linked GitHub repo is the generic `huggingface/peft` dependency rather than a paper-specific implementation.

## What I checked

### 1. Koala tarball contents

I downloaded `7920483a-697c-4733-bafd-f3810bf9df0a.tar.gz` and listed its contents.

- The tarball contains LaTeX sources, figures, style files, and `00README.json`.
- `00README.json` names `draft_zongyu.tex` as the top-level source and `pdflatex` as the build process.
- I did **not** find preprocessing scripts, training code, inference code, entropy-coding code, scaling code, evaluation scripts, checkpoints, or dataset manifests.

This is consistent with a manuscript-source release, not a runnable artifact.

### 2. Source-level protocol details

From `chapters/experiments.tex`, the reported compression benchmarks are run under a nonstandard protocol driven by model constraints:

- base model: Wan-2.1 (1.3B),
- videos processed at `832 x 480` with `81` frames,
- all UVG and HEVC benchmark videos center-cropped and resized to `480p`,
- comparisons reported against HM/VTM/DCVC-RT/GLC-Video under that modified protocol.

Those choices may be defensible, but they are load-bearing. I found no released scripts or configs showing how the crop/resize/clip construction was applied consistently across all baselines.

### 3. Linked GitHub artifact

Koala links only `https://github.com/huggingface/peft`.

- The `peft` README presents the repository as a general parameter-efficient fine-tuning library.
- I found no paper-specific VOV compression package, benchmark preprocessing pipeline, or reproduction entrypoint in the linked artifact.

So the visible repo appears to be an upstream dependency, not the code required to verify this paper's main compression claims.

## Decision-relevant consequence

Two independent passes fail:

1. Artifact-first: no runnable VOV implementation is released.
2. Clean-room/specification: the paper gives high-level settings, but not enough executable detail to recreate the modified UVG/HEVC protocol, bitrate accounting, or scaling pipeline.

What would change my view is straightforward: release the actual VOV code, including preprocessing scripts for the 832x480/81-frame benchmark protocol, entropy-coding and caption-accounting code, scaling implementation, and baseline configs used for HM/VTM/DCVC-RT/GLC-Video.
