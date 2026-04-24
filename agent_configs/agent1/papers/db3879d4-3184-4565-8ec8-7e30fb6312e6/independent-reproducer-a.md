# Independent Reproducer A Report

Paper: `db3879d4-3184-4565-8ec8-7e30fb6312e6`, "Self-Supervised Flow Matching for Scalable Multi-Modal Synthesis"

Role: Independent Reproducer A. I did not read Independent Reproducer B's report. Evidence was restricted to the Koala-provided paper artifacts under `papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts`, linked repos under `papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos`, and paper-internal prior-work references. I did not use OpenReview reviews, decisions, citation counts, social media, or later commentary.

## Claim Attempted

The central claim I attempted to reproduce was that Self-Flow, using Dual-Timestep Scheduling (DTS) and an EMA-teacher representation loss, improves generation and representation learning versus vanilla flow matching, SRA, REPA, and external-encoder variants across image, video, audio, and multimodal settings.

Full empirical reproduction was infeasible from the released artifacts. I therefore attempted the smallest meaningful checks:

1. Verify whether the official artifacts contain executable Self-Flow training/evaluation code, configs, checkpoints, raw generated samples, reference statistics, logs, or data splits sufficient to reproduce the reported tables and figures.
2. Inspect whether the linked `flux2` repo contains the paper's claimed per-token vector timestep conditioning and Self-Flow loss path.
3. Reproduce a minimal mathematical property of DTS: per-token timesteps preserve the marginal distribution while the teacher input is cleaner in expectation.
4. Recompute arithmetic deltas from the reported paper tables to verify that the paper's directional improvement claims are internally consistent.

## Setup Used

Working directory:

```text
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Environment observed:

```text
Linux cn-f004.server.mila.quebec 5.15.0-173-generic #183-Ubuntu SMP Fri Mar 6 13:29:34 UTC 2026 x86_64 GNU/Linux
Python 3.12.12
python3 -c "import torch" -> ModuleNotFoundError: No module named 'torch'
pdflatex -> /home/mila/l/lia/.TinyTeX/bin/x86_64-linux/pdflatex
pdftotext/pdfinfo -> not installed
```

Repo revisions:

```text
flux2 HEAD: 50fe5162777813d869182b139e83b10743caef15
guided-diffusion HEAD: 22e0df8183507e13a7813f8d38d51b072ca1e67c
```

I did not install dependencies or run GPU training because the provided environment lacks PyTorch and, more importantly, the artifacts lack the Self-Flow training code, datasets, checkpoints, sample archives, and evaluation reference statistics needed for a claim-level run.

## Artifact Completeness Check

Commands:

```bash
find papers/db3879d4-3184-4565-8ec8-7e30fb6312e6 -maxdepth 3 -type f | sort
tar -tzf papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/source.tar.gz | sort | head -n 240
find papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/figures -type f | sort
find papers/db3879d4-3184-4565-8ec8-7e30fb6312e6 -type f \( -iname '*.pt' -o -iname '*.pth' -o -iname '*.ckpt' -o -iname '*.safetensors' -o -iname '*.npz' -o -iname '*.npy' -o -iname '*.jsonl' -o -iname '*.csv' -o -iname '*.parquet' -o -iname '*.yaml' -o -iname '*.yml' \) -printf '%p\n' | sort
```

Observed:

- The paper bundle contains `paper.pdf`, `main.tex`, section `.tex` files, `example_paper.bib`, ICML style files, and figure PDFs.
- The source archive lists the same LaTeX and figure assets.
- The artifact includes many final figure PDFs, including `figures/t2i_fid.pdf`, `figures/fvd_figure.pdf`, `figures/fad_clap_ms.pdf`, `figures/ablation_figure.pdf`, `figures/scaling_clip_score_v5.pdf`, and multimodal/action plots.
- The searched model/data/log extensions returned only repo configuration files such as `repos/flux2/.github/workflows/ci.yaml` and style/config metadata. I found no training logs, raw metric tables, sample `.npz` files, generated-image batches, checkpoints, reference stats, data manifests, seeds, or complete experiment configs.

Conclusion: figure-source recoverability is only partial. The LaTeX source and final figure PDFs are present, but the numerical sources behind the plots are not. The reported metrics cannot be recomputed from the provided artifact bundle.

## Linked Repo Inspection

Commands:

```bash
rg --files papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos | sort
sed -n '1,240p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/README.md
sed -n '1,220p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/pyproject.toml
sed -n '1,220p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/guided-diffusion/README.md
rg -n -i "self[-_ ]?flow|dual[-_ ]?timestep|representation loss|l_rep|rep loss|repa|sra|masking ratio|mask_ratio|r_m|\bgamma\b|ema teacher|teacher network|cosine" papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts | head -n 240
find papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2 -type f \( -iname '*train*' -o -iname '*loss*' -o -iname '*config*' -o -iname '*.yaml' -o -iname '*.yml' \) -printf '%p\n' | sort
```

Observed:

- `repos/flux2` is a minimal FLUX.2 inference repository. Its README says it contains "minimal inference code to run image generation & editing with our FLUX.2 open-weight models." It does not describe Self-Flow training.
- `repos/flux2/pyproject.toml` depends on `torch==2.8.0`, `torchvision==0.23.0`, `transformers==4.56.1`, `safetensors==0.4.5`, and inference-related packages. It contains no training extras.
- `repos/flux2` has no training script and no Self-Flow, REPA, or SRA implementation file. The only train/config-like matches are CI and repository configuration files.
- `repos/guided-diffusion` is OpenAI guided-diffusion code. It is relevant to ImageNet FID evaluation only in the general sense noted by the paper appendix, but the repo does not contain this paper's generated samples or reference batches.
- String search for Self-Flow/DTS/representation-loss terms found the concepts in the paper source, not in executable linked code.

Most important implementation mismatch check:

```bash
sed -n '115,170p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/src/flux2/model.py
sed -n '260,410p' papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos/flux2/src/flux2/sampling.py
```

The paper states in Appendix "Architecture" that DTS extends timestep conditioning "from a single scalar `t in R^1` to a vector of timesteps `t in R^N` such that each token in the sequence is conditioned on its corresponding noising timestep." The released FLUX.2 inference path uses batch-level timestep vectors, not token-level timestep matrices. In `sampling.py`, denoising constructs:

```python
t_vec = torch.full((img.shape[0],), t_curr, dtype=img.dtype, device=img.device)
```

and passes that to `Flux2.forward`. In `model.py`, `Flux2.forward` embeds:

```python
timestep_emb = timestep_embedding(timesteps, 256)
vec = self.time_in(timestep_emb)
```

This is not the paper's DTS training path. I do not treat this as a code contradiction because the linked repo appears to be upstream FLUX.2 inference code rather than the authors' Self-Flow training implementation. It does mean that the official linked code does not reproduce the claimed method.

## Minimal DTS Mathematical Check

Paper claim checked: DTS samples two timesteps `t, s ~ p(t)`, assigns either `t` or `s` to tokens via a random mask, and "maintains the marginal timestep distribution per token"; the EMA teacher sees `tau_min = min(t, s)`, a cleaner input.

Command:

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
print('seed=20260424 trials=200000 N=256 R=0.25')
for name, arr in [('selected_tau', selected), ('unselected_tau', unselected), ('all_tau', all_tau), ('teacher_min', teacher_min)]:
    qs = statistics.quantiles(arr, n=4)
    print(f'{name}: mean={statistics.fmean(arr):.6f} q25={qs[0]:.6f} median={qs[1]:.6f} q75={qs[2]:.6f}')
print('expected Uniform mean for per-token tau = 0.5')
print('expected E[min(t,s)] for teacher cleaner input = 1/3')
PY
```

Observed output:

```text
seed=20260424 trials=200000 N=256 R=0.25
selected_tau: mean=0.500099 q25=0.250533 median=0.500046 q75=0.749655
unselected_tau: mean=0.499587 q25=0.248741 median=0.500340 q75=0.749281
all_tau: mean=0.499715 q25=0.249157 median=0.500248 q75=0.749361
teacher_min: mean=0.333243 q25=0.133903 median=0.292747 q75=0.499878
expected Uniform mean for per-token tau = 0.5
expected E[min(t,s)] for teacher cleaner input = 1/3
```

Result: match for the narrow mathematical property under uniform `p(t)`. This validates that the described DTS construction can preserve per-token timestep marginals while making the teacher cleaner in expectation. It does not validate the empirical generation or representation-learning claims.

## Reported Table Arithmetic Check

Command:

```bash
python3 - <<'PY'
comparisons = [
    ('ImageNet FID Ours vs REPA', 5.70, 5.89, 'lower'),
    ('ImageNet FID Ours vs SRA', 5.70, 7.27, 'lower'),
    ('RAE FID Ours vs RAE', 2.95, 3.24, 'lower'),
    ('T2I FID Ours vs SRA', 3.61, 3.70, 'lower'),
    ('T2I FID Ours vs REPA', 3.61, 3.92, 'lower'),
    ('T2I FD-DINO Ours vs REPA', 167.98, 173.35, 'lower'),
    ('Video FVD Ours vs SRA', 47.81, 49.75, 'lower'),
    ('Video FVD Ours vs DINOv2', 47.81, 49.59, 'lower'),
    ('Audio CLAP Ours vs SRA', 145.645, 147.215, 'lower'),
    ('Audio CLAP Ours vs Vanilla', 145.645, 148.874, 'lower'),
]
for name, ours, base, direction in comparisons:
    delta = ours - base
    rel = delta / base * 100
    better = delta < 0 if direction == 'lower' else delta > 0
    print(f'{name}: ours={ours} base={base} delta={delta:.3f} rel={rel:.2f}% better={better}')
PY
```

Observed output:

```text
ImageNet FID Ours vs REPA: ours=5.7 base=5.89 delta=-0.190 rel=-3.23% better=True
ImageNet FID Ours vs SRA: ours=5.7 base=7.27 delta=-1.570 rel=-21.60% better=True
RAE FID Ours vs RAE: ours=2.95 base=3.24 delta=-0.290 rel=-8.95% better=True
T2I FID Ours vs SRA: ours=3.61 base=3.7 delta=-0.090 rel=-2.43% better=True
T2I FID Ours vs REPA: ours=3.61 base=3.92 delta=-0.310 rel=-7.91% better=True
T2I FD-DINO Ours vs REPA: ours=167.98 base=173.35 delta=-5.370 rel=-3.10% better=True
Video FVD Ours vs SRA: ours=47.81 base=49.75 delta=-1.940 rel=-3.90% better=True
Video FVD Ours vs DINOv2: ours=47.81 base=49.59 delta=-1.780 rel=-3.59% better=True
Audio CLAP Ours vs SRA: ours=145.645 base=147.215 delta=-1.570 rel=-1.07% better=True
Audio CLAP Ours vs Vanilla: ours=145.645 base=148.874 delta=-3.229 rel=-2.17% better=True
```

Result: the table arithmetic is directionally consistent with the paper's claims. However, several central margins are modest, especially T2I FID versus SRA (-0.09 FID, -2.43%) and audio CLAP-FAD versus SRA (-1.07%). Without raw runs, seeds, confidence intervals, generated samples, or repeated evaluations, I cannot assess statistical robustness.

## Blockers

The core empirical claim is not independently reproducible from the available artifacts.

Concrete blockers:

- No Self-Flow training implementation was found in the linked repos.
- No code path implementing the paper's DTS vector timestep conditioning, EMA teacher feature extraction, projection-head representation loss, or combined `L_gen + gamma L_rep` training loop was found.
- No experiment configs for the 625M/1B models, ImageNet SiT-XL/RAE runs, T2I/T2V/T2A runs, multimodal weighting sweeps, or robotics finetuning runs were provided.
- No checkpoints, generated samples, raw metric feature statistics, evaluation manifests, data splits, seeds, or logs were provided.
- Key datasets are unavailable or internal: the paper describes an internal 200M image dataset, a 20M curated T2I subset, an internal 6M video dataset, and specific validation splits. These cannot be reconstructed from the artifacts.
- The paper's figure PDFs are final graphics, not raw data or plotting scripts.
- The environment lacks PyTorch, but this is a secondary issue; installing PyTorch would not solve the missing method implementation and data.

## Reproduction Outcome

Claim-level status: **blocked / weak reproducibility**.

Smallest-unit results:

- DTS marginal-distribution check: **match** for the narrow theoretical property under uniform timestep sampling.
- Reported table arithmetic: **match** directionally for selected main metrics.
- Artifact/code path existence for Self-Flow training: **blocked / missing**.
- Image/video/audio generation quality reproduction: **not reproducible** from provided artifacts.
- Representation learning claim, including linear probing: **not reproducible** from provided artifacts.
- Scaling-law claim versus REPA: **not reproducible** from provided artifacts.
- Multimodal and robotics transfer claims: **not reproducible** from provided artifacts.

## Confidence

I have high confidence that the supplied Koala artifacts are insufficient for independent empirical reproduction of the paper's central results. I have moderate confidence that the mathematical description of DTS is internally coherent at the marginal timestep level. I have low confidence in the empirical strength of the paper's core claims from my reproduction attempt, because I could only verify table arithmetic and artifact presence, not rerun any decisive experiment.

## Decision Impact

This should materially lower the reproducibility score. The paper may still be scientifically interesting, but its acceptance case rests on broad empirical claims across modalities and scales, using large internal datasets and unreleased training code. From a reproducibility-first review standard, the central claim remains unverified. I would not give strong acceptance credit for the reported image/video/audio/representation/scaling improvements unless another role obtains executable code, raw logs, checkpoints, or independently reproduced metrics from a faithful implementation.
