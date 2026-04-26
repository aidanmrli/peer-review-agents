# Reproducibility Lead Report

Paper: `7920483a-697c-4733-bafd-f3810bf9df0a`, "Compression as Adaptation: Implicit Visual Representation with Diffusion Foundation Models"

## Claim Tested

The load-bearing empirical claim is that VOV stores an 81-frame 832x480 video as a single length-131072 UniLoRA vector, entropy-codes it to roughly 1-3 bits per parameter, and obtains strong perceptual compression on UVG/HEVC against codecs such as VTM, HM, DCVC-RT, and GLC-Video, with further gains from inference-time scaling.

Relevant paper locations:

- `artifacts/chapters/experiments.tex:23-38`: VOV compression setting, datasets, and metrics.
- `artifacts/chapters/experiments.tex:93-104`: strong DISTS/FVD claim and caption/entropy overhead claim.
- `artifacts/chapters/experiments.tex:138-141`: two inference-time scaling budgets.
- `artifacts/chapters/appendix.tex:585-603`: stated compression hyperparameters: Wan-2.1-1.3B, rank 1, vector length 131072, lambda values, 1000+1000 iterations, batch size 64, 100 denoising steps, and 100/1000 scaling steps with `2^18/2^10` samples per step.

## Evidence Base

Artifacts inspected:

- Paper source: `papers/7920483a-697c-4733-bafd-f3810bf9df0a/artifacts/draft_zongyu.tex`
- Sections: `papers/7920483a-697c-4733-bafd-f3810bf9df0a/artifacts/chapters/`
- Official project page from paper `\paperlinks`: `https://compressionasadaptation.github.io/`
- Official project-page code link: `https://github.com/microsoft/VisionAsAdaptations`
- Local clone: `papers/7920483a-697c-4733-bafd-f3810bf9df0a/repos/VisionAsAdaptations`, commit `3d9091d52d49076459822a413a7b0cdd14d11b7e`
- Koala metadata repo: `https://github.com/huggingface/peft`, a generic PEFT library rather than the paper implementation.

Commands used:

```bash
curl -L -s https://compressionasadaptation.github.io/ | rg -n "github|Code|VisionAsAdaptations|href"
git clone --depth 1 https://github.com/microsoft/VisionAsAdaptations.git papers/7920483a-697c-4733-bafd-f3810bf9df0a/repos/VisionAsAdaptations
git rev-parse HEAD
rg --files papers/7920483a-697c-4733-bafd-f3810bf9df0a/artifacts
rg --files papers/7920483a-697c-4733-bafd-f3810bf9df0a/repos/VisionAsAdaptations
find repos/VisionAsAdaptations -maxdepth 4 -type f \( -name '*.safetensors' -o -name '*.pt' -o -name '*.pth' -o -name '*.ckpt' -o -name '*.json' -o -name '*.csv' -o -name '*.log' \) -print
bash -n image/train_multi.sh image/reconstruct_multi.sh video/train/stage1_train.sh video/train/stage2_train.sh video/train/run_two_stage.sh video/scaling/sample.sh video/eval/eval.sh
```

## Role Synthesis

- Independent Reproducer A: blocked from recovering the paper curves. The repo has training/reconstruction code and captions, but no dataset manifest, exact benchmark configs, released checkpoints, raw metrics, or baseline outputs.
- Independent Reproducer B: found a more direct codec gap. The scaling implementation computes a reference latent from original frames, saves selected indices, but provides no decoder-side replay script that consumes those transmitted indices without the original video.
- Implementation Auditor: official code exists and is relevant, but defaults and scripts diverge from the paper: stage-2 training defaults to one step, paper-setting batch size 64 is not encoded, `2^18` scaling candidates are not exposed, and no actual entropy-coded bitstream path is released.
- Correctness Specialist: the vector-size bitrate arithmetic is internally plausible, but the released scaling code does not instantiate the paper's encoder/decoder contract. It is an encoding experiment rather than a demonstrated decompression artifact.
- Literature Specialist: novelty is a real combination of LoRA/Uni-LoRA, INR compression, and diffusion/relative-entropy coding, but the contribution should be read as an application and systems integration rather than a fully specified portable codec.

## Reproducibility Assessment

Partial-to-weak reproducibility. A competent reviewer can inspect and likely run a single-sequence Wan/Qwen UniLoRA overfitting experiment after obtaining large models and frames, but cannot independently recover the central UVG/HEVC rate-distortion results from released materials. The missing pieces are not cosmetic: exact benchmark configs, checkpoints, raw metric tables, baseline command lines, entropy-coded bitstream generation, and decoder-side scaling replay are absent.

## Score Impact

The repository materially improves the paper relative to a no-code submission, but it does not support the strongest compression and inference-time-scaling claims at the standard expected for an empirical ML systems paper. Reproducibility should substantially downgrade the score unless the authors provide paper-setting configs, bitstreams/checkpoints, raw RD tables, and a true decoder implementation.
