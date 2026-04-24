# Reproducibility Lead Report

Paper ID: `db3879d4-3184-4565-8ec8-7e30fb6312e6`

Title: Self-Supervised Flow Matching for Scalable Multi-Modal Synthesis

Assigned role: Reproducibility Lead

Date: 2026-04-24

## Task Scope

I coordinated the internal review for `agent1` and checked whether the paper's core claims can be independently reproduced from the paper, Koala artifacts, linked repositories, and permitted prior literature.

The central claim tested is that Self-Flow combines Dual-Timestep Scheduling and an internal EMA-teacher representation loss to improve flow-matching generation and representation learning over vanilla flow matching, SRA, REPA, and external-encoder variants across image, video, audio, multimodal, and action-prediction settings.

I treated reproducibility as the dominant evaluation axis. A claim receives strong confidence only if at least two independent roles can reproduce or independently validate the core result.

## Evidence Examined

Local artifact directory:

```text
papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/
```

Key paper files:

- `artifacts/sec/4_method.tex`: Self-Flow and Dual-Timestep Scheduling definitions.
- `artifacts/sec/5_experiments.tex`: ImageNet, T2I, video, audio, scaling, ablation, multimodal, and action results.
- `artifacts/main.tex`: appendix details on datasets, architecture, hyperparameters, evaluation, mixed-modality batches, and RT-1/SIMPLER.

Linked repositories:

- `repos/flux2`, cloned from `https://github.com/black-forest-labs/flux2`, HEAD `50fe5162777813d869182b139e83b10743caef15`.
- `repos/guided-diffusion`, cloned from `https://github.com/openai/guided-diffusion`, HEAD `22e0df8183507e13a7813f8d38d51b072ca1e67c`.

Role reports:

- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

## Commands and Checks Used

Representative commands run by the team:

```bash
find papers/db3879d4-3184-4565-8ec8-7e30fb6312e6 -maxdepth 3 -type f | sort
tar -tzf papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/source.tar.gz | sort
find papers/db3879d4-3184-4565-8ec8-7e30fb6312e6 -type f \
  \( -iname '*.pt' -o -iname '*.pth' -o -iname '*.ckpt' -o -iname '*.safetensors' \
     -o -iname '*.npz' -o -iname '*.npy' -o -iname '*.csv' -o -iname '*.jsonl' \
     -o -iname '*.yaml' -o -iname '*.yml' \) -printf '%p\n' | sort
rg -n -i "self[-_ ]?flow|dual[-_ ]?timestep|representation_loss|teacher|student|FVD|FAD|CLAP|train|eval" \
  papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2 \
  papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/guided-diffusion \
  --glob '!**/.git/**'
sed -n '1,240p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/README.md
sed -n '260,410p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/src/flux2/sampling.py
sed -n '700,730p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/src/flux2/model.py
nl -ba papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/sec/4_method.tex
nl -ba papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/sec/5_experiments.tex
nl -ba papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/main.tex | sed -n '152,242p'
```

Clean-room schedule check used by the two reproducers:

```bash
python3 - <<'PY'
import random, statistics
random.seed(20260424)
N = 256
R = 0.25
trials = 200000
selected = []
unselected = []
all_tau = []
teacher_min = []
for _ in range(trials):
    t = random.random()
    s = random.random()
    tau_min = min(t, s)
    teacher_min.append(tau_min)
    for _ in range(N):
        if random.random() < R:
            tau = s
            selected.append(tau)
        else:
            tau = t
            unselected.append(tau)
        all_tau.append(tau)
print('mean all tau', statistics.fmean(all_tau))
print('mean teacher min', statistics.fmean(teacher_min))
PY
```

Result: per-token `tau` has mean about 0.5 under uniform timestep sampling, while `min(t, s)` has mean about 1/3. This validates the narrow marginal-timestep and cleaner-teacher property, but not the empirical claims.

## Role-by-Role Findings

### Independent Reproducer A

Reproducer A attempted artifact-level and code-path reproduction. The central empirical result could not be rerun. The paper bundle contains LaTeX, final figures, and bibliography files, but no Self-Flow training code, configs, logs, checkpoints, generated samples, reference statistics, data manifests, seeds, or table source data. A reproduced DTS schedule sanity check matched the paper's marginal-timestep claim. A table arithmetic check confirmed that selected reported metric deltas are directionally consistent. This is weak reproducibility for the central claim.

### Independent Reproducer B

Reproducer B used an independent audit route through Koala metadata, LaTeX tables, linked repositories, method equations, and a pure-Python schedule check. This role also reproduced only the narrow schedule property and table transcription. It found that `flux2` is minimal inference code for public FLUX.2 models, while `guided-diffusion` provides ADM-era ImageNet evaluation/training code. Neither repository contains Self-Flow training, Dual-Timestep Scheduling training, EMA teacher loss, paper-specific configs, sample batches, checkpoints, or multimodal metric pipelines.

### Implementation Auditor

The implementation audit found critical reproducibility blockers. The linked `flux2` repository does not implement Self-Flow or Dual-Timestep Scheduling; its README describes "minimal inference code" and its sampling path constructs one timestep per batch element. The public model configs are 32B, 9B, and 4B FLUX.2 product configs, not the paper's 290M/420M/625M/1B experimental backbones. The `guided-diffusion` repository matches only the paper's ImageNet evaluator citation and requires `.npz` sample/reference batches that are not provided. No code covers FD-DINO, CLIP, FVD, CLAP-FAD, linear probes, SIMPLER, or paper-specific training.

### Correctness Specialist

The correctness pass found major technical weaknesses in the written method and interpretation. Dual-Timestep Scheduling preserves per-token timestep marginals, but not the joint distribution used at inference. With `N=256` and `R_M=0.25`, the probability of a fully homogeneous training state is about `(0.75)^256 + (0.25)^256 = 1.04e-32`, while inference uses homogeneous scalar-time states. The paper does not define a scalar flow-matching objective for vector-timestep states, so the connection between off-diagonal training and the diagonal inference ODE is underspecified. The masking ratio is not the fraction of high-noise tokens when `t` and `s` are unordered. Some metric deltas are small and lack uncertainty; audio hyperparameters appear selected on the reported metric family.

### Literature Specialist

The literature pass found that Self-Flow has a real distinction from REPA and SRA if the empirical results reproduce: it adapts internal EMA teacher-student alignment to mixed-timestep flow-matching states. However, the broad novelty framing is overstated. Relevant prior work includes SRA, LayerSync, Diffuse and Disperse, SD-DiT, MaskDiT/MDT, Diffusion Forcing, BYOL/DINO-style self-distillation, diffusion-as-representation work, and unified multimodal generation systems such as Transfusion, JanusFlow, Show-o, Chameleon, and VideoPoet. Missing or underused baselines include Dispersive Loss and cross-modal LayerSync. Literature impact is score-limiting rather than fatal by itself.

## Reproduction Outcome

Two independent reproducer roles agree:

- Core empirical generation and representation claims: not independently reproduced.
- Self-Flow training implementation: not found in official linked repositories.
- Dual-Timestep marginal timestep sanity check: reproduced.
- Cleaner EMA-teacher input under `min(t, s)`: reproduced.
- Table arithmetic and transcription of selected metrics: partially checked.
- Image/video/audio/multimodal/action metrics: blocked.
- Scaling and ablation curves: blocked.
- Linear probes and representation-learning claims: blocked.

Classification: weak reproducibility for the central empirical claims.

## Exact Reproducibility Blockers

- No Self-Flow training code or paper-specific experiment repository.
- No implementation of Dual-Timestep Scheduling training, vector timestep conditioning, EMA teacher/student feature extraction, projection heads, or `L_gen + gamma * L_rep`.
- No configs for ImageNet SiT/RAE runs, FLUX-style 625M/1B runs, T2I/T2V/T2A runs, multimodal sampling, or RT-1/SIMPLER finetuning.
- No checkpoints, EMA weights, optimizer states, generated samples, raw logs, `.npz` sample batches, reference stats, figure-data files, seeds, or table-regeneration scripts.
- Major datasets are internal or underspecified: internal 200M images with 20M curated T2I subset, internal 6M videos, FMA-derived captioned audio subset, and RT-1/SIMPLER preprocessing.
- Non-ImageNet metric pipelines are absent for FD-DINO, CLIP, VideoMAEv2 FVD, CLAP-FAD variants, linear probes, and SIMPLER.

## Score Impact

The paper is ambitious and the method is plausible as a heuristic. However, the acceptance case rests on broad empirical superiority across modalities and scales, exactly where the artifacts are weakest. Under `agent1`'s reproducibility-first standard, this should materially cap the score.

Recommended verdict range if no additional evidence appears: weak reject to low weak accept, approximately 4.0 to 5.5. I would not support a strong accept unless paper-specific code, configs, checkpoints, data manifests or sample batches, and metric pipelines become available or other independent agents reproduce the central results.

## Final Synthesis

The internal team did not reproduce the central empirical claim. Both independent reproducer roles validated only small, local properties of the schedule and tables. The linked repositories do not implement the method described by the paper. The written method also has a technical gap between vector-timestep training states and scalar-time inference. The literature contribution is meaningful but narrower than stated. The correct public comment should therefore be a high-confidence reproducibility warning: promising idea, but the current paper package does not support the reported 9/10-level confidence in empirical superiority.
