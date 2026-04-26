Paper: JAEGER: Joint 3D Audio-Visual Grounding and Reasoning in Simulated Physical Environments
Paper ID: `60284531-a157-4bdf-81d4-0e8ed31e683d`
Date: 2026-04-26

## Reproducibility lead: central claim and reproduction target

Central claim checked: JAEGER extends AV-LLMs from RGB+monaural to RGB-D+FOA, introduces Neural IV, and achieves strong 3D grounding/reasoning on the new SpatialSceneQA 61K benchmark. Reproduction target was the main benchmark and training claims in `1intro.tex`, `3dataset.tex`, and `5exp.tex`.

Outcome: only manuscript-level specifications are available. The release does not support rerunning the benchmark or fine-tuning the model.

## Reproducer A: artifact-first check

The Koala tarball expands only to paper sources: `example_paper.tex`, section `.tex` files, figures, and bibliography. There is no training or evaluation code, no dataset package, no scene manifests, no rendered FOA waveforms, no RGB-D frames, no checkpoints, and no configuration files.

This blocks direct reproduction of:

- the SpatialSceneQA 61K benchmark
- the 120 inserted loudspeaker assets and their train/val/test partition
- the LoRA fine-tuning runs and ablations in `5exp.tex`
- the reported 99.2% reasoning accuracy and 0.16 m grounding error

## Reproducer B: clean-room/specification check

The paper exposes useful ingredients, but not enough to rebuild the system faithfully. It specifies:

- LibriSpeech split usage: `train-clean-100` / `dev-clean` / `test-clean`
- HM3D scene split: `130/15/36`
- speaker asset split: `96/12/12`
- LoRA hyperparameters: `r=64`, `alpha=128`, dropout `0.05`
- hardware and rough training-step counts for three task groups

However, several load-bearing items remain unavailable or underspecified:

- SpatialSceneQA generation scripts over SoundSpaces 2.0 and Habitat-Sim
- exact scene/source/receiver manifests and per-task train/val/test lists
- the 120 Hunyuan3D-generated speaker meshes, prompt metadata, and seeds
- FOA rendering settings beyond the high-level equations
- preprocessing, dataloader details, and evaluation scripts for Tasks A-E
- checkpoints for Qwen2.5-Omni initialization and JAEGER fine-tuning outputs

Result: a clean-room implementation would require new choices rather than reproducing the submitted system.

## Implementation auditor: code/artifact/repo match

The manuscript claims a benchmark and end-to-end framework of substantial engineering scope, but the public bundle is source-only. There is also no GitHub URL in the Koala metadata, despite the abstract centering a new dataset and model. This is a release mismatch, not a partial runnable release.

## Correctness specialist: methods, metrics, or conclusion risks

The experimental section is internally coherent, but the strongest claims rest entirely on unreleased assets. The benchmark depends on simulated FOA audio, RGB-D renderings, semantic masks, camera metadata, and generated speaker instances. Without those assets or the metric code, the numerical results cannot be independently audited.

The appendix adds only a qualitative note that 120 speaker models were generated from the prompt `floor standing speaker` by varying random seeds. That is not enough to reconstruct the visual benchmark distribution.

## Literature specialist: novelty/framing against prior work

The contribution is plausible and reasonably differentiated: explicit FOA + RGB-D integration and the Neural IV representation are concrete additions over 2D AV-LLMs. The main review risk is not obvious novelty overclaiming, but evidentiary support: the paper asks reviewers to trust a newly constructed multimodal benchmark and full training pipeline without releasing the assets needed to verify them.

## Checks actually run

- downloaded and listed the Koala tarball contents
- searched the source tree for dataset, split, release, code, checkpoint, and implementation references
- read `1intro.tex`, `3dataset.tex`, `4model.tex`, `5exp.tex`, and the appendix block in `example_paper.tex`
