# Implementation Auditor Report

## Claim Tested

Whether the released implementation supports the paper's central compression and scaling claims, including training, inference, evaluation/benchmark scripts, configs, checkpoints/weights, runtime artifacts, hard-coded paths, dependency pinning, and paper-code alignment.

## Repository Status

Koala metadata points to `https://github.com/huggingface/peft`, but the paper source links `https://compressionasadaptation.github.io/` (`artifacts/draft_zongyu.tex:165`), and that project page links `https://github.com/microsoft/VisionAsAdaptations`. I cloned the latter at commit `3d9091d52d49076459822a413a7b0cdd14d11b7e`.

The official repo is relevant and contains:

- `image/train_lora_multi.py`, `image/reconstruct_lora_multi.py`, `image/unilora_utils.py`, and Qwen pipeline code.
- `video/train/stage1_vm/train_lora_single.py` and `video/train/stage2_vs/train_lora_single.py`.
- `video/scaling/reconstruct_lora_single_scaling_encode.py` and `video/scaling/pipeline_wan_scaling_encode.py`.
- `video/eval/eval_metrics.py`, `video/eval/prepare_fvd_clips.py`, and `video/eval/compute_fvd.py`.
- per-sequence captions for 19 UVG/HEVC-style names under `video/**/descriptions/`.

## Training Inventory

The code implements a plausible two-stage training pipeline matching the paper at a high level:

- Stage 1 trains a UniLoRA vector without entropy constraint: `video/train/stage1_train.sh`.
- Stage 2 initializes from stage 1 and adds a rate loss: `video/train/stage2_train.sh` and `video/train/stage2_vs/train_lora_single.py`.
- The flow-matching loss is implemented as `F.mse_loss(pred_v, noise - latents)` at `video/train/stage2_vs/train_lora_single.py:418-426`, matching `artifacts/chapters/method_zongyu.tex:22-35`.
- UniLoRA hashing is implemented with deterministic PRNG indices in `video/train/stage2_vs/unilora_utils.py:14-24` and assigned to LoRA A/B matrices at `video/train/stage2_vs/unilora_utils.py:129-158`.
- Rate estimation is implemented in `video/train/stage2_vs/unilora_utils.py:220-260`.

Paper-code mismatches:

- Paper compression Table: 1000 stage-1 + 1000 stage-2 iterations and batch size 64 (`artifacts/chapters/appendix.tex:589-594`).
- Repo `video/train/stage1_train.sh:26` defaults to `TRAIN_STEPS=2000`.
- Repo `video/train/stage2_train.sh:25` defaults to `TRAIN_STEPS=1`.
- Repo `video/train/stage1_train.sh:15-17` and `video/train/stage2_train.sh:11-13` default to 2 GPUs, batch size 1, grad accumulation 2, effective batch 4, not paper batch 64.
- The scripts are runnable only after overriding many cluster paths, e.g. `/data1/Compression_clean/...`, `/output/datasets/...`, `/data1/VISUAL_SUBMIT/...` in `video/train/stage1_train.sh:8-25`, `video/train/stage2_train.sh:7-24`, and `video/scaling/sample.sh:8-21`.

Severity: high. The implementation is not packaged with exact paper configs, and the README examples will not run paper-equivalent experiments by default.

## Inference and Scaling Inventory

Base reconstruction is present for trained weights. Scaling is implemented in `video/scaling/pipeline_wan_scaling_encode.py:646-701`.

Critical gaps:

- `video/scaling/reconstruct_lora_single_scaling_encode.py:694-706` loads a reference latent from original frames and passes it into inference.
- `video/scaling/pipeline_wan_scaling_encode.py:646-647` computes the target score from `reference_latent`.
- `video/scaling/reconstruct_lora_single_scaling_encode.py:599-605` saves `best_idx`, but there is no corresponding loader/replay decoder path.
- `video/scaling/pipeline_wan_scaling_encode.py:667-668` hard-codes `2**10` candidates. The paper reports a `2^18` candidates-per-step setting.

Severity: high. The released scaling path is an encoder-side experiment, not a complete compression decoder implementation.

## Evaluation and Benchmark Inventory

Evaluation scripts exist:

- PSNR/DISTS/LPIPS: `video/eval/eval_metrics.py:39-125`.
- FVD clip creation: `video/eval/prepare_fvd_clips.py:28-110`.
- FVD computation: `video/eval/compute_fvd.py:29-52` and `video/eval/compute_fvd.py:122-180`.

Missing benchmark artifacts:

- No raw UVG/HEVC metric tables.
- No exact list of clips/windows/crops.
- No baseline command lines for VTM/HM/DCVC-RT/GLC-Video.
- No decoded output directories, compressed bitstreams, or result JSONs corresponding to the paper figures.

Severity: high for independent verification of the main curves.

## Checkpoints, Weights, and Bitstreams

I searched for `.safetensors`, `.ckpt`, `.pt`, `.pth`, `.json`, `.csv`, and `.log` artifacts under the repo. Aside from GitHub workflow files, the release does not include checkpoints, model-output logs, raw metric JSON/CSV files, or compressed bitstreams. Large base models are expected from Hugging Face (`Wan-AI/Wan2.1-T2V-1.3B-Diffusers`, `Qwen/Qwen-Image`), but no exact revision hashes are specified in the scripts.

The paper claims caption and entropy-parameter overhead are included in curves and below 1% (`artifacts/chapters/experiments.tex:104`). The repo has captions, but no code path or artifact proving those overheads were added to the plotted RD tables.

Severity: high.

## Dependency Pinning

Dependencies are only partially pinned:

- `video/requirements.txt` uses broad lower bounds such as `diffusers>=0.30.0`, `torch>=2.2.0`, `transformers>=4.44.0,<5`.
- `video/train/stage2_vs/requirements.txt` pins `diffusers==0.35.1` and `numpy<2`, but leaves PyTorch, PEFT, transformers, and CUDA stack floating.
- The code depends on large diffusion model internals and copied pipeline files, so dependency drift can plausibly affect reproducibility.

Severity: medium.

## Paper-Code Alignment Summary

The repo supports the general idea: frozen Wan/Qwen backbones, UniLoRA one-vector adaptation, two-stage rate-regularized training, and metric scripts. It does not support the paper's central reported performance as a fully reproducible codec. Exact paper configs, outputs, checkpoints, bitstreams, and a decoder-side scaling replay path are absent.

Overall severity for acceptance: high.
