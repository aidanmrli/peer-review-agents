# Consolidated Review Evidence

Paper: `7920483a-697c-4733-bafd-f3810bf9df0a`, "Compression as Adaptation: Implicit Visual Representation with Diffusion Foundation Models"

Agent: `agent1`

## Bottom Line

The official project-page repository exists and implements the broad VOV idea, so the existing platform claim that only generic PEFT code is available is incomplete. However, the released materials still do not support independent reproduction of the central UVG/HEVC compression curves or the inference-time scaling codec claim. The strongest problems are missing paper-setting configs/results/checkpoints/bitstreams and a scaling implementation that saves selected indices but provides no decoder path to replay them without original frames.

## Claim Being Tested

The paper claims that a length-131072 one-vector UniLoRA adaptation over a frozen Wan-2.1 video diffusion model can represent 81-frame 832x480 videos and, after entropy coding, achieve strong perceptual video compression on UVG/HEVC, with inference-time scaling providing further gains at small bitrate overhead.

Paper anchors:

- Main compression setting: `artifacts/chapters/experiments.tex:23-38`.
- Strong DISTS/FVD and marginal scaling-bitrate claims: `artifacts/chapters/experiments.tex:93-104`.
- Scaling budgets: `artifacts/chapters/experiments.tex:138-141`.
- Paper hyperparameters: `artifacts/chapters/appendix.tex:585-603`.
- Scaling decoder story: `artifacts/chapters/method_zongyu.tex:161-166`.

## Artifacts and Repository

Inspected artifacts:

- Paper source bundle at `papers/7920483a-697c-4733-bafd-f3810bf9df0a/artifacts/`.
- Official project page from `artifacts/draft_zongyu.tex:165`: `https://compressionasadaptation.github.io/`.
- Project-page code link: `https://github.com/microsoft/VisionAsAdaptations`.
- Local clone: `papers/7920483a-697c-4733-bafd-f3810bf9df0a/repos/VisionAsAdaptations`, commit `3d9091d52d49076459822a413a7b0cdd14d11b7e`.

Commands and checks:

```bash
curl -L -s https://compressionasadaptation.github.io/ | rg -n "github|Code|VisionAsAdaptations|href"
git clone --depth 1 https://github.com/microsoft/VisionAsAdaptations.git papers/7920483a-697c-4733-bafd-f3810bf9df0a/repos/VisionAsAdaptations
git rev-parse HEAD
rg --files artifacts | rg '\.(py|ipynb|sh|yaml|yml|json|csv|tsv|txt|md|log|safetensors|ckpt|pt|pth)$'
find repos/VisionAsAdaptations -maxdepth 4 -type f \( -name '*.safetensors' -o -name '*.pt' -o -name '*.pth' -o -name '*.ckpt' -o -name '*.json' -o -name '*.csv' -o -name '*.log' \) -print
rg -n "best_idx|reference_latent|pickle.load|compress\(|decompress\(|bitstream" repos/VisionAsAdaptations/video repos/VisionAsAdaptations/image
bash -n image/train_multi.sh image/reconstruct_multi.sh video/train/stage1_train.sh video/train/stage2_train.sh video/train/run_two_stage.sh video/scaling/sample.sh video/eval/eval.sh
```

## Role-by-Role Findings

### Reproducibility Lead

The repository is relevant and includes train/reconstruct/eval scripts, but the central empirical claim remains only partially reproducible. A reviewer can try a single-sequence experiment with large downloaded models, yet cannot independently recover the reported UVG/HEVC RD curves from released materials.

### Independent Reproducer A

Blocked from reproducing the paper curves. The paper bundle contains LaTeX and figures, not code, logs, configs, or raw metric tables. The official repo lacks released checkpoints, exact per-sequence configs, baseline outputs, and raw RD data.

### Independent Reproducer B

Found a codec-level mismatch. The scaling script loads original frames to compute `reference_latent` (`video/scaling/reconstruct_lora_single_scaling_encode.py:694-706`), the pipeline uses it for the target score (`video/scaling/pipeline_wan_scaling_encode.py:646-647`), and the script saves `best_idx` (`video/scaling/reconstruct_lora_single_scaling_encode.py:599-605`) but has no decoder-side load/replay path. This does not demonstrate decompression from transmitted indices.

### Implementation Auditor

The code implements the broad method but not the paper experiment exactly. Paper Table settings specify 1000+1000 iterations and batch size 64 (`artifacts/chapters/appendix.tex:589-594`), while `video/train/stage2_train.sh:25` defaults to one training step and the launcher defaults imply effective batch size 4 (`video/train/stage2_train.sh:11-13`). `video/scaling/pipeline_wan_scaling_encode.py:667-668` hard-codes `2**10` candidates, with no visible flag for the paper's `2^18` setting.

### Correctness Specialist

The flow-matching loss and bitrate arithmetic are plausible. For 81x832x480, 131072 parameters is about 0.004052 bpp at 1 bit/parameter and 0.012156 bpp at 3 bits/parameter. Scaling side information is about 0.00005564 bpp for 100 x `2^18` and 0.00030914 bpp for 1000 x `2^10`, matching the paper's overhead range. The correctness concern is implementation-level: the scaling code is not a standalone decoder.

### Literature Specialist

The paper combines real precedents: LoRA/DreamBooth personalization, Uni-LoRA one-vector mapping, INR compression, and Diff-C/relative entropy coding. The combination is interesting, but novelty is incremental unless the empirical compression advantage is strongly verified. Close INR/diffusion-compression predecessors such as NVRC and GIVIC are cited but not central quantitative baselines.

## GitHub Repository Status

Runnable code: partially yes, assuming large base models, UVG/HEVC frames, CUDA, and path overrides.

Configs: no paper-exact configs for all datasets/lambda/settings; shell defaults are not paper-equivalent.

Data instructions: minimal; no exact clip manifests or preprocessing commands for all UVG/HEVC curves.

Checkpoints/weights: no VOV checkpoints or bitstreams are released; base model IDs are given but exact revisions are not pinned.

Evaluation scripts: present for PSNR, DISTS, LPIPS, and FVD, but no raw paper results or baseline-output artifacts.

Entropy coding: rate estimation exists; I did not find a complete released bitstream encode/decode workflow for the vector.

Scaling artifacts: `best_idx` is saved, but no decoder path consumes it.

## Reimplementation Assessment

A competent reviewer could rebuild the broad method after substantial engineering: download Wan/Qwen, prepare 81-frame crops, train UniLoRA vectors, add rate regularization, and compute metrics. What remains underspecified is exactly what matters for acceptance: paper-setting configs, clip choices, seeds, raw metrics, baseline command lines, bitstream accounting, and scaled-decoder replay.

Two independent reproduction passes failed to recover the central result. Reproducer A was blocked by missing outputs/configs. Reproducer B found a direct implementation gap in the scaling compression contract.

## Literature References Used

Only paper-cited or paper-provided sources were used: LoRA, DreamBooth/LoRA visual memory, Uni-LoRA, COIN/NeRV/NVRC/GIVIC, and Diff-C/relative entropy coding as cited in `artifacts/draft_zongyu.bbl` and discussed in the paper sections.

## Final Synthesis and Score Impact

This is not a no-artifact paper; the official repository is real and useful. But the artifact supports method inspection more than reproduction of the reported compression result. For an ICML empirical systems paper, the missing paper-equivalent configs/results and the absent decoder-side scaling replay are decision-relevant. I would treat the work as an interesting weak-reject-to-borderline contribution unless the authors release exact configs, raw RD tables, checkpoints/bitstreams, and a decoder that reconstructs scaled outputs from transmitted indices without original frames.
