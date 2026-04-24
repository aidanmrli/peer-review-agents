# Independent Reproducer B Report

Paper: `db3879d4-3184-4565-8ec8-7e30fb6312e6`, "Self-Supervised Flow Matching for Scalable Multi-Modal Synthesis"  
Role: Independent Reproducer B for `agent1`  
Date: 2026-04-24  
Independence note: I did not read `independent-reproducer-a.md` before writing this report.

## Claim Attempted

I tested the central claim that Self-Flow, using Dual-Timestep Scheduling plus an internal EMA-teacher representation loss, can be independently specified and reproduced well enough to substantiate the reported improvements over vanilla flow matching, SRA, and external-alignment baselines across image, video, audio, and multi-modal synthesis.

My route deliberately avoided a full training rerun and instead used an alternate audit path: LaTeX source tables and figures, linked repository code paths, method equations, ablations, evaluation details, and a small clean-room schedule sanity check.

## Sources Used

Permitted sources only:

- Koala paper metadata from `get_paper`.
- Paper PDF and source tarball from Koala storage.
- Linked GitHub repositories from the Koala metadata:
  - `https://github.com/black-forest-labs/flux2`
  - `https://github.com/openai/guided-diffusion`
- Local role instruction `skills/independent-reproducer-b.md`.
- Live Koala platform guide `https://koala.science/skill.md`.

I did not use OpenReview, citation counts, external commentary, acceptance status, or post-publication signals.

## Environment and Commands

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Key commands run:

```bash
curl -fsSL https://koala.science/skill.md
sed -n '1,260p' skills/independent-reproducer-b.md
```

Koala metadata retrieved with `get_paper`:

```text
title: Self-Supervised Flow Matching for Scalable Multi-Modal Synthesis
status: in_review
pdf_url: /storage/pdfs/db3879d4-3184-4565-8ec8-7e30fb6312e6.pdf
tarball_url: /storage/tarballs/db3879d4-3184-4565-8ec8-7e30fb6312e6.tar.gz
github_urls:
  https://github.com/black-forest-labs/flux2
  https://github.com/openai/guided-diffusion
```

Artifacts were copied only to scratch space:

```bash
rm -rf /tmp/koala-db3879d4
mkdir -p /tmp/koala-db3879d4/source /tmp/koala-db3879d4/repos
curl -fsSL https://koala.science/storage/pdfs/db3879d4-3184-4565-8ec8-7e30fb6312e6.pdf \
  -o /tmp/koala-db3879d4/paper.pdf
curl -fsSL https://koala.science/storage/tarballs/db3879d4-3184-4565-8ec8-7e30fb6312e6.tar.gz \
  -o /tmp/koala-db3879d4/source.tar.gz
tar -xzf /tmp/koala-db3879d4/source.tar.gz -C /tmp/koala-db3879d4/source
find /tmp/koala-db3879d4/source -maxdepth 3 -type f | sort
```

Repository commits inspected:

```bash
git ls-remote https://github.com/black-forest-labs/flux2 HEAD refs/heads/main
git ls-remote https://github.com/openai/guided-diffusion HEAD refs/heads/main
git clone --depth 1 https://github.com/black-forest-labs/flux2 /tmp/koala-db3879d4/repos/flux2
git clone --depth 1 https://github.com/openai/guided-diffusion /tmp/koala-db3879d4/repos/guided-diffusion
```

Observed commits:

```text
black-forest-labs/flux2: 50fe5162777813d869182b139e83b10743caef15
openai/guided-diffusion: 22e0df8183507e13a7813f8d38d51b072ca1e67c
```

Local tooling limitations:

```bash
compgen -c | rg '^(pdftotext|pdfinfo|mutool|qpdf|gs|inkscape|pdfimages|python|convert)$' | sort -u
```

Output:

```text
python
```

The active Python environment did not have `numpy` or `torch`, so executable model-level shape tests were blocked:

```text
ModuleNotFoundError: No module named 'numpy'
ModuleNotFoundError: No module named 'torch'
```

## Paper and Artifact Locations Checked

Main method:

- `sec/4_method.tex:6-23`: rectified-flow objective and interpolation.
- `sec/4_method.tex:81-99`: Dual-Timestep Scheduling definition.
- `sec/4_method.tex:103-120`: Self-Flow EMA-teacher representation loss.

Main quantitative claims:

- `sec/5_experiments.tex:12-35`: ImageNet 256x256 table. Ours reports FID 5.70 vs REPA 5.89, and RAE + Ours reports FID 2.95 vs RAE 3.24.
- `sec/5_experiments.tex:59-79`: T2I table. Ours reports FID 3.61, FD-DINO 167.98, CLIP 30.88.
- `sec/5_experiments.tex:82-102`: video table. Ours reports FVD 47.81 and framewise FID 8.92.
- `sec/5_experiments.tex:107-127`: audio table. Ours reports best FAD/CLAP variants.
- `sec/5_experiments.tex:187-209`: scaling claims.
- `sec/5_experiments.tex:272-307`: ablation claims.

Implementation and evaluation details:

- `main.tex:155-163`: datasets, including internal 200M-image and 6M-video research datasets.
- `main.tex:221-227`: autoencoders and timestep distributions by modality.
- `main.tex:231-234`: token-vector timestep conditioning, architecture, EMA, gamma, layer ratios, mask ratios, CFG/evaluation distinction.
- `main.tex:236-242`: evaluation procedures and metric feature sources.
- `main.tex:390-398`: multi-modal batch sizes, sampling ratios, and modality weights.
- `main.tex:436-440`: joint video-action setup.
- `main.tex:479-506`: ImageNet convergence and semantic-autoencoder supplemental claims.
- `main.tex:532-537`: layer-selection ablation explanation.

Linked code:

- `flux2/README.md:13`: repository says it contains "minimal inference code".
- `flux2/README.md:91-93`: `FLUX.2 [dev]` is a 32B model requiring H100-equivalent VRAM.
- `flux2/src/flux2/model.py:10-49`: public configs are 32B/9B/4B FLUX.2 variants, not the paper's 625M experimental architecture.
- `flux2/src/flux2/model.py:115-134`: forward path consumes a `timesteps` tensor and immediately makes one modulation vector.
- `flux2/src/flux2/model.py:710-725`: `timestep_embedding` documents `t` as one-dimensional, "one per batch element".
- `guided-diffusion/evaluations/README.md` and `evaluator.py`: ImageNet FID/sFID/IS/precision/recall evaluation code exists, but no paper sample batches are provided.

## Clean-Room Schedule Sanity Check

The paper claims that Dual-Timestep Scheduling preserves the per-token marginal timestep distribution while creating information asymmetry between the mixed-noise student view and the cleaner EMA-teacher view.

Relevant equations:

- Sample two timesteps `t, s ~ p(t)`.
- Assign each token `tau_i = s` if masked, else `tau_i = t`.
- Teacher view uses `tau_min = min(t, s)`.
- Student input is `x_tau = diag(1 - tau) x_0 + diag(tau) x_1`.

I checked the uniform-timestep special case with a pure-Python simulation because `numpy` was unavailable:

```bash
python - <<'PY'
import random, bisect, statistics
rng=random.Random(20260424)
N=200000
R=0.25
t=[rng.random() for _ in range(N)]
s=[rng.random() for _ in range(N)]
mask=[rng.random()<R for _ in range(N)]
tau=[s[i] if mask[i] else t[i] for i in range(N)]
tau_min=[t[i] if t[i] < s[i] else s[i] for i in range(N)]
xs=[i/1000 for i in range(1001)]
stau=sorted(tau)
stmin=sorted(tau_min)
ks_tau=max(abs(bisect.bisect_right(stau,x)/N-x) for x in xs)
ks_min=max(abs(bisect.bisect_right(stmin,x)/N-(1-(1-x)**2)) for x in xs)
mt=statistics.fmean(tau); mm=statistics.fmean(tau_min)
num=sum((a-mt)*(b-mm) for a,b in zip(tau,tau_min))
den=(sum((a-mt)**2 for a in tau)*sum((b-mm)**2 for b in tau_min))**0.5
print('seed=20260424 N=200000 R=0.25')
print('mean_t={:.6f} mean_s={:.6f} mean_tau={:.6f} mean_tau_min={:.6f}'.format(
    statistics.fmean(t),statistics.fmean(s),mt,mm))
print('ks_tau_vs_uniform={:.6f}'.format(ks_tau))
print('ks_tau_min_vs_min_analytic={:.6f}'.format(ks_min))
print('corr_tau_tau_min={:.6f}'.format(num/den))
print('frac_teacher_cleaner_than_student={:.6f}'.format(
    sum(1 for a,b in zip(tau,tau_min) if b<a)/N))
PY
```

Observed output:

```text
seed=20260424 N=200000 R=0.25
mean_t=0.501777 mean_s=0.500533 mean_tau=0.501343 mean_tau_min=0.334381
ks_tau_vs_uniform=0.003285
ks_tau_min_vs_min_analytic=0.002401
corr_tau_tau_min=0.610465
frac_teacher_cleaner_than_student=0.498775
```

Status: match for the stated schedule math. The student marginal remains essentially uniform, and the teacher's `min(t, s)` distribution is substantially cleaner. This supports the internal consistency of the schedule definition, but it does not reproduce the reported empirical improvements.

Important nuance: because the paper samples unordered `t` and `s`, the smaller masked subset is not always the noisier subset. If `s > t`, the masked tokens are noisier; if `t > s`, the unmasked tokens are noisier. The method still creates asymmetry, but "masking ratio" should not be read as a fixed "noisier-token ratio" unless the implementation orders the two timesteps before assignment. I found no released training implementation to verify that detail.

## LaTeX Table and Figure Audit

The source tarball contains the LaTeX tables and compiled figure PDFs, but no raw experiment logs, CSVs, configs, generated samples, checkpoints, or scripts for recomputing the plotted curves.

Command:

```bash
find /tmp/koala-db3879d4/source -type f \
  \( -name '*.csv' -o -name '*.json' -o -name '*.npy' -o -name '*.npz' -o -name '*.py' -o -name '*.sh' -o -name '*.yaml' -o -name '*.yml' -o -name '*.txt' \) \
  -printf '%P\n' | sort
```

Output:

```text
00README.json
```

The figure files exist only as one-page PDFs. Representative command:

```bash
find /tmp/koala-db3879d4/source/figures -type f -name '*.pdf' -printf '%P %s bytes\n' | sort | sed -n '1,120p'
file /tmp/koala-db3879d4/source/figures/ablation_figure.pdf \
     /tmp/koala-db3879d4/source/figures/scaling_clip_score_v5.pdf \
     /tmp/koala-db3879d4/source/figures/repa_scaling_v2.pdf \
     /tmp/koala-db3879d4/source/figures/noise_scheduling_v2.pdf \
     /tmp/koala-db3879d4/source/figures/mixed/radar.pdf
```

Observed result: the source includes many PDFs such as `ablation_figure.pdf`, `scaling_clip_score_v5.pdf`, `repa_scaling_v2.pdf`, `noise_scheduling_v2.pdf`, and `mixed/radar.pdf`, each reported by `file` as `PDF document, version 1.4, 1 pages`. Without `pdftotext`, `qpdf`, `mutool`, or raw plotting data, I could verify that the paper contains the figures but could not independently recover the numeric points behind scaling, ablation, radar, or convergence plots.

Status: partial match for table transcription; blocked for independent figure-data reproduction.

## Linked Repository Audit

The `flux2` repository does not appear to contain the code needed to reproduce Self-Flow training or evaluation.

Command:

```bash
rg -n "train|Self-Flow|self flow|dual|timestep|noise|flow|loss|representation|DINO|FID|FVD|CLAP|ImageNet|Wan|Songbloom|autoencoder|checkpoint|weights|eval|sampling|scheduler|mask" \
  /tmp/koala-db3879d4/repos/flux2 -g '!*.jpg' -g '!*.png'
```

Observed result:

- The README explicitly describes the repository as "minimal inference code" (`README.md:13`).
- The public model configs are for 32B, 9B, and 4B FLUX.2 models, while the paper's main non-ImageNet experiments use a roughly 625M FLUX-style architecture with hidden size 1152, 7 double blocks, and 14 single blocks (`main.tex:231-234`).
- I found no Self-Flow training loop, no Dual-Timestep Scheduling implementation, no representation loss implementation, no SRA/REPA comparison code, no released configs for the 625M experimental model, no metric scripts for FVD/FAD/CLAP, and no sample batches.
- The public `timestep_embedding` path is documented as one timestep per batch element (`model.py:710-717`), whereas the paper says Dual-Timestep Scheduling extends conditioning from scalar `t` to a vector of timesteps per token (`main.tex:231`). This is a paper-code mismatch for the released repository, or at minimum evidence that the linked repository is not the experimental code used for the paper.

The `guided-diffusion` repository only covers the ImageNet-style evaluation code cited by the paper. It can compute metrics if supplied reference and sample `.npz` files, but the paper does not provide generated sample batches or checkpoints. Thus even the ImageNet headline table cannot be regenerated from the supplied artifacts.

Status: mismatch/block. The linked code is insufficient for reproducing the central empirical claims.

## Reproducibility Blockers

End-to-end empirical reproduction is blocked by missing or inaccessible components:

- No released Self-Flow training code.
- No implementation of Dual-Timestep Scheduling and the EMA-teacher representation loss in the linked repository.
- No configs for the reported 625M architecture, optimizer, learning-rate schedule, weight decay, warmup, global batch for single-modality training, seeds, mixed precision, data shuffling, or distributed setup.
- No checkpoints for Ours, vanilla flow, SRA, REPA, or external-encoder variants.
- No generated sample batches for ImageNet/T2I/video/audio.
- No raw plotting data for scaling, ablations, noise-scheduler impact, layer ablations, multi-modal radar plots, or robotics success curves.
- Major datasets are internal: the paper reports a 200M-image research dataset, a 20M curated T2I subset, and a 6M-video research dataset (`main.tex:158-161`). These cannot be independently reconstructed from the paper.
- FMA audio data is public/CC-licensed in principle, but the exact captioning, filtering, split, preprocessing, and generated sample set are not released.
- Non-ImageNet evaluation depends on modality-specific validation data and feature extractors, but no ready-to-run scripts or feature caches are supplied.

## Match / Partial / Mismatch Summary

| Check | Status | Evidence |
|---|---|---|
| Dual-Timestep marginal schedule math | Match | Pure-Python simulation confirms `tau` preserves the timestep marginal and `tau_min` is cleaner. |
| Self-Flow equation readability | Match | Method equations are clear enough to reconstruct symbolically from `sec/4_method.tex:81-120`. |
| Tables in LaTeX source | Partial | Reported headline numbers are present in source tables, but this only verifies transcription, not correctness. |
| Figure/ablation/scaling reproduction | Blocked | Only compiled PDFs are supplied; no raw plotted data or scripts. |
| Linked FLUX.2 code reflects Self-Flow method | Mismatch | Repository is minimal inference code and lacks Self-Flow training/evaluation paths. |
| ImageNet evaluation code availability | Partial | Guided-diffusion evaluation code exists, but no sample batches/checkpoints are supplied. |
| Multi-modal empirical reproduction | Blocked | Internal datasets and unreleased training/evaluation code dominate the evidence. |
| Central empirical claim independently reproduced | Blocked | I could not reproduce the reported improvements across image/video/audio/multi-modal tasks. |

## Agreement With Reproducer A

Not assessed in this file. I did not read Reproducer A's report before finishing, per instruction. Comparison should be performed only after both independent reports are complete.

## Final Synthesis and Score Impact

Independent Reproducer B can validate the internal schedule math and can verify that the claimed numerical results are written into the LaTeX tables, but cannot independently reproduce the paper's central empirical claims from the released artifacts. The linked FLUX.2 repository is an inference repository for public FLUX.2 models, not the Self-Flow experimental codebase, and the source tarball lacks raw data, scripts, configs, checkpoints, and generated samples. The strongest claims--superior T2I/T2V/T2A metrics, scaling behavior, ablation effects, and multi-modal/robotics gains--therefore remain unsupported by independent reproduction through this route.

Reproduction outcome: weak reproducibility for the central empirical claims; match only for the schedule sanity check.  
Confidence: high for the artifact-gap conclusion; medium-high for the mathematical schedule check; low for empirical claims because they are blocked rather than contradicted.  
Review consequence: this should materially lower confidence in the paper despite the plausible method definition and strong reported tables. The paper may still be scientifically interesting, but under agent1's reproducibility-first standard, the core acceptance case is not reproducible from the provided paper, artifacts, and linked repositories.
