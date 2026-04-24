# Implementation Auditor Report

Paper: `db3879d4-3184-4565-8ec8-7e30fb6312e6`, "Self-Supervised Flow Matching for Scalable Multi-Modal Synthesis"  
Role: Implementation Auditor  
Audit date: 2026-04-24  
Scope: static artifact and repository audit only. I did not use citation counts, stars, reviews, decisions, OpenReview discussion, or post-release impact signals.

## Paper Claim Being Tested

The implementation claim under audit is that the official artifacts support the paper's core method and experiments:

- Self-Flow training: an EMA teacher and student trained with `L_gen + gamma * L_rep`.
- Dual-Timestep Scheduling: two timesteps sampled per example, token mask `M`, vector timestep `tau in R^N`, heterogeneous noising, and cleaner teacher input at `tau_min`.
- Multi-modal experiments over image, video, audio, mixed multi-modal training, and video-action prediction.
- Evaluation metrics and reported numbers: ImageNet FID/sFID/IS/Precision/Recall, T2I FD-DINO and CLIP, video FVD/FID, audio FAD with CLAP variants, linear probes, SIMPLER success rates, ablations, scaling runs, and checkpoints/configs sufficient to reproduce them.

## Artifact Inventory

Platform metadata reports two official GitHub URLs:

- `https://github.com/black-forest-labs/flux2`
- `https://github.com/openai/guided-diffusion`

Local official artifact directory inspected:

- `artifacts/paper.pdf`
- `artifacts/source.tar.gz`
- LaTeX source: `artifacts/main.tex`, `artifacts/sec/*.tex`, `artifacts/example_paper.bib`
- Figures only under `artifacts/figures/**`
- No experiment scripts, training configs, model checkpoints, sample batches, metric logs, Docker/conda environment, or run commands are present in the paper artifact bundle.

The source tarball inventory is LaTeX and figures only. A local search for model/checkpoint/config/sample artifacts found only:

```text
./artifacts/00README.json
./repos/flux2/.pre-commit-config.yaml
./repos/flux2/.yamlfmt.yaml
./repos/flux2/pyproject.toml
```

No `.pt`, `.pth`, `.safetensors`, `.npz`, `.yaml`, `.yml`, training shell script, or paper-specific JSON config exists locally for Self-Flow.

Repository commit state:

```text
repos/flux2
origin: https://github.com/black-forest-labs/flux2
HEAD: 50fe5162777813d869182b139e83b10743caef15
branch: main
last commit: 2026-03-12T16:00:27+01:00, "FLUX.2 [klein] KV"
shallow clone: true
tracked files at HEAD: 31

repos/guided-diffusion
origin: https://github.com/openai/guided-diffusion
HEAD: 22e0df8183507e13a7813f8d38d51b072ca1e67c
branch: main
last commit: 2022-07-15T12:24:59-07:00, "fix for fp16 loss scaling (#52)"
shallow clone: true
tracked files at HEAD: 30
```

## Commands And Code Paths Inspected

Representative commands run:

```bash
rg -n "Self-Flow|Dual-Timestep|FLUX|guided-diffusion|FID|FVD|FAD|CLAP|training|checkpoint" artifacts/main.tex artifacts/sec/*.tex
git -C repos/flux2 rev-parse HEAD
git -C repos/guided-diffusion rev-parse HEAD
git -C repos/flux2 ls-tree -r --name-only HEAD
git -C repos/guided-diffusion ls-tree -r --name-only HEAD
rg -n -i "self[-_ ]?flow|dual[-_ ]?timestep|representation_loss|teacher|student|FVD|FAD|CLAP|train|eval" repos/flux2 repos/guided-diffusion --glob '!**/.git/**'
find . -maxdepth 4 -type f \( -iname '*ckpt*' -o -iname '*.pt' -o -iname '*.pth' -o -iname '*.safetensors' -o -iname '*.npz' -o -iname '*.yaml' -o -iname '*.yml' -o -iname '*.json' -o -iname '*.sh' -o -iname '*.toml' \)
```

Key inspected paper locations:

- `artifacts/sec/4_method.tex:73-120`: Dual-Timestep Scheduling and Self-Flow definition.
- `artifacts/main.tex:152-242`: implementation details, datasets, autoencoders, timestep distributions, architecture, evaluation.
- `artifacts/sec/5_experiments.tex:166-185`: main single-modality experiment setup and reported image/video/audio metrics.
- `artifacts/sec/5_experiments.tex:207-209`: scaling experiments.
- `artifacts/main.tex:392-396`: mixed-modality batch sizes, sampling ratios, and loss weights.
- `artifacts/main.tex:436-440`: RT-1/SIMPLER action-prediction setup.

Key inspected repository paths:

- `repos/flux2/README.md`
- `repos/flux2/pyproject.toml`
- `repos/flux2/scripts/cli.py`
- `repos/flux2/src/flux2/model.py`
- `repos/flux2/src/flux2/sampling.py`
- `repos/flux2/src/flux2/util.py`
- `repos/guided-diffusion/README.md`
- `repos/guided-diffusion/evaluations/evaluator.py`
- `repos/guided-diffusion/evaluations/README.md`
- `repos/guided-diffusion/scripts/image_train.py`
- `repos/guided-diffusion/guided_diffusion/train_util.py`
- `repos/guided-diffusion/guided_diffusion/gaussian_diffusion.py`
- `repos/guided-diffusion/guided_diffusion/resample.py`

## Paper-To-Code Matches

There are only narrow, non-novel matches.

1. `flux2` is plausibly relevant to the architecture family and autoencoder cited by the paper. The paper states that non-ImageNet experiments are based on the FLUX architecture with FLUX.2 changes and uses the FLUX.2 autoencoder for some image/multi-modal settings. The `flux2` repo contains a FLUX.2 transformer, autoencoder loader, sampling utilities, and public model loading from Hugging Face.

2. `guided-diffusion` matches the paper's ImageNet evaluation footnote only. The paper says ImageNet scores are computed using `openai/guided-diffusion/tree/main/evaluations` and the ADM reference batch. The local `repos/guided-diffusion/evaluations/evaluator.py` computes Inception Score, FID, sFID, Precision, and Recall from `.npz` sample/reference batches.

3. The `flux2` environment is pinned for public inference: `pyproject.toml` lists Python `>=3.10,<3.13`, `torch==2.8.0`, `torchvision==0.23.0`, `transformers==4.56.1`, `safetensors==0.4.5`, `fire==0.7.1`, `openai==2.8.1`, and `accelerate==1.12.0`. This is useful for running the public FLUX.2 inference CLI, not for reproducing the paper's training.

## Paper-To-Code Discrepancies

### 1. No Self-Flow training implementation is released

The paper defines Self-Flow as an EMA teacher/student representation objective (`artifacts/sec/4_method.tex:103-120`) and gives training constants in `artifacts/main.tex:231-233`: `gamma=0.8`, student layer `0.3D`, teacher layer `0.7D`, EMA `0.9999`, and modality-specific second-timestep ratios.

The released `flux2` repo has no training loop, optimizer, data loader, loss implementation, EMA teacher training path, feature projection head, representation loss, layer extraction for `L_rep`, or paper-specific experiment config. Its README explicitly describes the repository as "minimal inference code to run image generation & editing with our FLUX.2 open-weight models"; the only script is `scripts/cli.py`, which calls `model.eval()`, `ae.eval()`, and `text_encoder.eval()` for generation.

The `guided-diffusion` repo has an ADM-era diffusion training loop, but that loop optimizes a standard diffusion loss over a single sampled timestep per image. It is not a flow-matching transformer, not Self-Flow, not a FLUX.2 backbone, and not multi-modal.

### 2. Dual-Timestep Scheduling is not implemented in the released code

The paper's core method requires a per-token timestep vector `tau in R^N` (`artifacts/sec/4_method.tex:81-99`; `artifacts/main.tex:231`). The public `flux2` code does not construct two timesteps, a token mask, heterogeneous token noising, or teacher/student timestep inputs.

Evidence:

- `repos/flux2/src/flux2/sampling.py:283-301` constructs `t_vec = torch.full((img.shape[0],), t_curr, ...)`, a single scalar timestep per batch element during inference.
- `repos/flux2/src/flux2/model.py:710-718` documents `timestep_embedding` as accepting "a 1-D Tensor of N indices, one per batch element"; the implementation uses `t[:, None]`, consistent with batch-level scalar timesteps rather than token-level `R^N` timesteps.
- No search hit for `Self-Flow`, `Dual-Timestep`, `representation_loss`, `R_M`, `tau_min`, or equivalent paper-specific terms exists in either repo.

The `guided-diffusion` training path similarly samples a single timestep per microbatch example (`train_util.py:189`) and computes `diffusion.training_losses(model, micro, t, ...)` (`train_util.py:191-214`). It has no token-level heterogeneous noising, no flow velocity objective, and no teacher representation loss.

### 3. Official `flux2` configs do not match the paper's reported 625M/290M/420M/1B experimental configurations

The paper's non-ImageNet 625M configuration is `hidden_size=1152`, `mlp_ratio=4`, `num_heads=16`, `7` double MMBlocks, and `14` single Blocks (`artifacts/main.tex:232-233`). Scaling runs use depths `8`, `14`, `21`, and `28` for 290M, 420M, 625M, and 1B models (`artifacts/sec/5_experiments.tex:207-209`).

The public `flux2` repository defines public product model configurations instead:

- `Flux2Params`: `hidden_size=6144`, `num_heads=48`, `depth=8`, `depth_single_blocks=48`
- `Klein9BParams`: `hidden_size=4096`, `num_heads=32`, `depth=8`, `depth_single_blocks=24`
- `Klein4BParams`: `hidden_size=3072`, `num_heads=24`, `depth=5`, `depth_single_blocks=20`

There is no checked-in config for the paper's 625M FLUX.2-like backbone, the reported scaling sweep, the SiT-XL ImageNet setup, RAE setup, or any Self-Flow variant.

### 4. No experiment datasets or data processing pipelines are released

The paper relies heavily on non-public or underspecified datasets:

- Internal 200M image dataset with 20M curated T2I subset and four captions per image (`artifacts/main.tex:158-159`).
- Internal 6M video dataset with multiple visual/audio captions and a 5k validation set (`artifacts/main.tex:161`).
- 1M 10-second FMA-derived audio samples with captions and a 20k validation split (`artifacts/main.tex:163`).
- RT-1/SIMPLER action-prediction preprocessing and evaluation (`artifacts/main.tex:436-440`).

No released repo contains the data manifests, filtering rules, caption selection logic, train/validation split IDs, preprocessing scripts, modality batch sampler, per-modality loss-weighting implementation, or evaluation prompts. The paper gives some high-level constants, but not enough to independently reconstruct the datasets or exact evaluation sets.

### 5. No paper checkpoints or generated sample batches are released

Neither local repo contains paper-trained weights, EMA checkpoints, optimizer states, logs, sample `.npz` batches, generated image/video/audio samples, linear-probe features, or validation activations. The `flux2` code can download public FLUX.2 product weights from Hugging Face, but those are not identified as the paper's Self-Flow checkpoints and do not expose the paper's training state or ablations.

The absence is decisive for auditing reported metrics. `guided-diffusion/evaluations/evaluator.py` can compute ImageNet metrics if given reference and sample `.npz` files, but the paper supplies no sample batches for Self-Flow, REPA, SRA, ablations, or scaling runs.

### 6. Evaluation code coverage is incomplete

Only ImageNet-style Inception metrics are covered by the linked `guided-diffusion/evaluations` code. The paper's central non-ImageNet claims require additional metric pipelines:

- FD-DINO and CLIP for text-to-image.
- VideoMAEv2 FVD and framewise Inception FID for video.
- CLAP, CLAP-M, and CLAP-A FAD for audio.
- Linear probing of intermediate representations.
- SIMPLER simulation success-rate evaluation.

No corresponding implementation, environment, model versions, command lines, or cached reference statistics are present in either official repo.

### 7. Exact commands are missing

The paper provides descriptive hyperparameters but no executable commands for:

- Self-Flow training.
- Vanilla flow matching, REPA, SRA, LayerSync baselines.
- ImageNet SiT-XL and RAE experiments.
- T2I/T2V/T2A training.
- Mixed-modality training with batch probabilities and loss weights.
- Scaling runs.
- Linear probing.
- Evaluation and table/figure regeneration.

The only executable command in `flux2` is an interactive public inference CLI (`PYTHONPATH=src python scripts/cli.py`). The only guided-diffusion commands are for ADM diffusion training/sampling/evaluation, not for this paper's method.

## Environment And Dependency Findings

- `flux2` provides a pinned inference environment and says it was tested on GB200 with CUDA 12.9 and Python 3.12. This is not a training environment for Self-Flow.
- `guided-diffusion` has minimal unpinned training dependencies (`blobfile>=1.0.5`, `torch`, `tqdm`) and an evaluation requirements file with `tensorflow-gpu>=2.0`, `scipy`, `requests`, `tqdm`. This older evaluator may be sufficient for ADM-style ImageNet metrics, but it does not cover the paper's non-ImageNet metrics.
- No Dockerfile, conda file, Slurm script, seed file, hardware accounting script, or distributed training launch file is provided for the reported large-scale experiments.

## Reproducibility Blockers

Critical blockers:

- Missing implementation of the central method: Self-Flow and Dual-Timestep Scheduling are not present in the official linked repos.
- Missing paper-specific training code, configs, hyperparameter files, and exact commands.
- Missing checkpoints, EMA weights, optimizer states, generated sample batches, and metric logs.
- Missing data manifests and splits for the internal 200M image dataset, internal 6M video dataset, FMA captioned subset, and RT-1/SIMPLER setup.
- Missing metric pipelines for FD-DINO, CLIP, VideoMAEv2 FVD, CLAP-based FAD, linear probes, and SIMPLER success rates.

Major blockers:

- The public `flux2` model configs do not match the paper's reported 625M/290M/420M/1B configurations.
- The public `flux2` code is an inference repository for released product models, not an experiment repository for the submitted method.
- The public `guided-diffusion` repository is a baseline/evaluation repository for the 2021 ADM paper; it is relevant only to ImageNet metric computation, not to Self-Flow training.
- The paper references a supplementary website for videos/audio/images, but the artifact source inspected here does not include a concrete supplementary website URL or downloadable sample package.

## Severity For Acceptance Decision

Severity: critical for reproducibility, high for empirical credibility.

The official artifacts do not implement the paper's central algorithm. They also do not provide the datasets, checkpoints, configs, commands, sample batches, or metric pipelines needed to reproduce the decisive empirical claims. The two linked repositories are best characterized as:

- `black-forest-labs/flux2`: relevant public FLUX.2 inference/model-loading code and autoencoder reference, not the Self-Flow/Dual-Timestep training implementation.
- `openai/guided-diffusion`: an external ADM baseline/evaluation repository, relevant only to ImageNet-style FID/sFID/Precision/Recall computation once sample batches already exist.

This audit does not prove the method is wrong, but it establishes that the reported core results are not independently reproducible from the official artifacts. The acceptance score should be materially downgraded on reproducibility grounds unless another internal role finds an official, paper-specific implementation or complete reproduction package outside the inspected artifacts.

## Final Synthesis

The repositories do not support an implementation-level verification of Self-Flow. There is no code-paper match for Dual-Timestep Scheduling, the self-supervised representation loss, EMA teacher/student training, multi-modal training, reported checkpoints, or most evaluation metrics. The only reliable match is that the paper cites `guided-diffusion` for ImageNet evaluation code, and that the paper borrows architectural/autoencoder ideas from public FLUX.2. For a reproducibility-first review, the correct conclusion is that the implementation artifacts are insufficient and the central empirical claims remain unverified.
