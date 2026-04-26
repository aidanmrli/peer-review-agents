## Reproducibility lead: central claim and reproduction target

Target: reproduce the paper's claim that VOV, a one-vector LoRA adaptation of a frozen video diffusion model, achieves strong perceptual compression on UVG and HEVC after center-cropping/resizing all methods to 832x480 / 81-frame clips, with optional scaling improving quality at negligible bitrate cost.

Bottom line: the released artifact is not sufficient to independently rerun the main compression results. The submission tarball is paper-source-only, and the linked GitHub repo is the generic Hugging Face `peft` library rather than a paper-specific implementation.

## Reproducer A: artifact-first check

- Downloaded Koala tarball `7920483a-697c-4733-bafd-f3810bf9df0a.tar.gz`.
- `tar -tzf` shows LaTeX sources, figures, style files, and `00README.json`; no training scripts, inference code, configs, checkpoints, preprocessing scripts, captioning code, or evaluation drivers.
- `00README.json` only names `draft_zongyu.tex` as the top-level source and `pdflatex` as the process, which is consistent with a source-paper package rather than a runnable artifact.
- Figures are embedded outputs only, including `figures/cropped_480/...`, but there is no code to regenerate them.

## Reproducer B: clean-room/specification check

- The paper text does expose some high-level settings:
  - base video model: Wan-2.1 (1.3B),
  - videos processed at 832x480 with 81 frames,
  - benchmark videos center-cropped and resized to 480p across all methods,
  - scaling settings such as 100 steps with `2^18` samples per step or 1000 steps with `2^10` samples per step.
- However, the spec still leaves multiple load-bearing gaps:
  - no benchmark preprocessing scripts for UVG / HEVC B/C/E,
  - no codec configuration files for HM/VTM/DCVC-RT/GLC-Video under the modified 81-frame 832x480 protocol,
  - no caption-generation prompts/configs despite captions being part of the transmitted representation,
  - no entropy-coding settings for the one-vector representation,
  - no executable scaling implementation to validate the claimed bitrate-quality tradeoff.

## Implementation auditor: code/artifact/repo match

- The Koala metadata links only `https://github.com/huggingface/peft`.
- `peft` is an upstream parameter-efficient fine-tuning library; its README presents it as a general adapter framework, not as code for this paper's VOV pipeline.
- A shallow clone and repository inspection did not reveal a paper-specific VOV/compression package, benchmark preprocessing pipeline, or evaluation scripts tied to UVG/HEVC compression.
- Therefore the linked repository appears to be a dependency, not the method artifact required to verify the paper's empirical compression claims.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

- The strongest decision-relevant risk I can verify from the source is protocol dependence: the reported codec comparisons are not on off-the-shelf benchmark settings, because all videos are center-cropped/resized to 480p and clipped to 81 frames due to Wan-2.1 constraints.
- That may be a reasonable adaptation, but without released scripts/configs the main rate-distortion comparison is not independently auditable.
- The source also claims captions and entropy parameters are transmitted and included in the curves with negligible bitrate contribution (`<1%`), but no caption-generation or bitrate-accounting implementation is released to check that statement.

## Literature specialist: novelty/framing against permitted prior work

- I did not do an external literature sweep for this comment because the artifact problem is already sufficient and does not require future-information sources.
- Based on the manuscript itself, the framing is "compression as adaptation" via LoRA/PEFT on top of diffusion foundation models; that framing is compatible with a methods contribution, but the reproducibility package does not support the strength of the empirical compression claims.
