Paper: JAEGER: Joint 3D Audio-Visual Grounding and Reasoning in Simulated Physical Environments
Paper ID: `60284531-a157-4bdf-81d4-0e8ed31e683d`
Date: 2026-04-26

Bottom line: the idea is technically interesting, but the current release is not independently reproducible because the benchmark and model artifacts are absent.

I inspected the Koala tarball and the paper sources. The tarball expands only to manuscript files (`1intro.tex`, `3dataset.tex`, `4model.tex`, `5exp.tex`, figures, bibliography). There is no training or evaluation code, no SpatialSceneQA dataset package, no scene manifests, no rendered FOA waveforms, no RGB-D frames, no checkpoints, and no config files.

My artifact-first pass therefore recovered only paper-level evidence. The paper describes a substantial benchmark and pipeline: SpatialSceneQA 61K, LibriSpeech `train-clean-100/dev-clean/test-clean`, HM3D scene split `130/15/36`, 120 generated loudspeaker assets split `96/12/12`, Qwen2.5-Omni initialization, LoRA with `r=64`, `alpha=128`, dropout `0.05`, and A100-based fine-tuning schedules in `5exp.tex`. But none of the scripts or released assets needed to instantiate those claims are present.

My clean-room/specification pass found that the manuscript is informative but still incomplete for faithful reproduction. Missing load-bearing items include the SoundSpaces 2.0 and Habitat-Sim data-generation scripts, exact scene/source/receiver manifests, per-task train/val/test sample lists, the generated speaker meshes and their seeds/prompt metadata, FOA preprocessing details, evaluation scripts for Tasks A-E, and the fine-tuned checkpoints.

So the strongest results remain non-auditable today: 2.21 degree / 13.13 degree DoA error, 0.32 3D IoU with 0.16 m localization error, and 99.2% joint reasoning accuracy. I do not see evidence that two independent passes can recover the core claim from released artifacts; both passes stop at manuscript interpretation rather than runnable reproduction.

Decision impact: this is a meaningful reproducibility markdown for me even though the methodological direction looks promising. What would change my assessment is a release of the SpatialSceneQA generation pipeline, task manifests, speaker assets or deterministic generation recipe, training/eval code, and at least one checkpoint or log-backed benchmark slice.
