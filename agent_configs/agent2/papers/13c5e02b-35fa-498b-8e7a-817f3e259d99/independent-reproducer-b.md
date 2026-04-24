# Independent Reproducer B Report

Paper: 13c5e02b-35fa-498b-8e7a-817f3e259d99
Title: "UniDWM: Towards a Unified Driving World Model via Multifaceted Representation Learning"
Role: Independent Reproducer B
Date: 2026-04-24

## Claim Attempted

I attempted to independently validate the central empirical and methodological claim that UniDWM's multifaceted latent representation, learned through joint reconstruction and collaborative generation, improves NAVSIM trajectory planning and 4D reconstruction while providing a VAE/InfoVAE-style theoretical basis.

This pass deliberately avoided the likely command-path reproduction route because the linked implementation artifact was not available: the author-stated repository `https://github.com/Say2L/UniDWM` had already been attempted and failed with public 404/credential prompt. I therefore used a separate route: source-level reconstruction of the reported objectives plus manual recomputation and consistency checks from the reported tables and appendix.

## Independent Route Used

Artifacts inspected:

- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/paper.pdf`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/0_abstract.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/1_intro.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/3_method.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/4_exp.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/00README.json`

Main route:

1. Reconstructed the objective derivation from the appendix and method text.
2. Recomputed reported table arithmetic for 4D reconstruction, planning gains, ablation gains, GRPO gains, and smoothness-analysis deltas.
3. Checked whether the table values support the paper's decision-relevant claims without relying on unavailable code.
4. Recorded blockers that prevent full empirical reproduction.

## Commands and Calculations

Role instructions:

```bash
sed -n '1,220p' skills/independent-reproducer-b.md
```

Artifact/source inspection:

```bash
find papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts -maxdepth 3 -type f | sort
sed -n '1,320p' papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/3_method.tex
sed -n '1,360p' papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/4_exp.tex
sed -n '260,470p' papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex
```

Manual arithmetic check:

```bash
awk 'BEGIN {
print "Table 3r overall from VGGT:", (3.764+2.236)/2;
print "Table 3r overall from Spann3R:", (2.028+2.202)/2;
print "Table 3r overall from UniDWM:", (1.878+1.575)/2;
print "Overall reduction vs VGGT:", (3.000-1.727)/3.000*100;
print "Overall reduction vs Spann3R:", (2.115-1.727)/2.115*100;
print "PDMS DINOv3-B minus Epona:", 90.6-86.2;
print "PDMS DINOv3-B minus DINOv3 baseline:", 90.6-85.4;
print "PDMS DINOv3-B minus World4Drive:", 90.6-85.1;
print "DCAE vs ablation+GRPO all row:", 84.9-84.9;
print "Ablation appearance gain:", 81.3-78.5;
print "Ablation geometry-only gain:", 80.0-78.5;
print "Ablation dynamic-only gain:", 80.9-78.5;
print "Ablation geom+dyn gain:", 81.6-78.5;
print "Full all gain:", 82.4-78.5;
print "GRPO baseline gain:", 81.2-78.5;
print "GRPO UniDWM gain:", 84.9-82.4;
print "Smoothness kNN reduction pct:", (130.578-117.238)/130.578*100;
print "Smoothness Lap reduction pct:", (328596-263981)/328596*100;
print "Smoothness PCA increase:", 0.622-0.554;
}'
```

Observed output:

```text
Table 3r overall from VGGT: 3
Table 3r overall from Spann3R: 2.115
Table 3r overall from UniDWM: 1.7265
Overall reduction vs VGGT: 42.4333
Overall reduction vs Spann3R: 18.3452
PDMS DINOv3-B minus Epona: 4.4
PDMS DINOv3-B minus DINOv3 baseline: 5.2
PDMS DINOv3-B minus World4Drive: 5.5
DCAE vs ablation+GRPO all row: 0
Ablation appearance gain: 2.8
Ablation geometry-only gain: 1.5
Ablation dynamic-only gain: 2.4
Ablation geom+dyn gain: 3.1
Full all gain: 3.9
GRPO baseline gain: 2.7
GRPO UniDWM gain: 2.5
Smoothness kNN reduction pct: 10.2161
Smoothness Lap reduction pct: 19.664
Smoothness PCA increase: 0.068
```

## Observed Result

### 1. The reported 4D reconstruction arithmetic is internally consistent.

Table `tab:3r` reports VGGT `Acc.=3.764`, `Comp.=2.236`, `Overall=3.000`; Spann3R `2.028`, `2.202`, `2.115`; UniDWM `1.878`, `1.575`, `1.727` in `sec/4_exp.tex` lines 47-60. Recomputing `Overall=(Acc.+Comp.)/2` gives:

- VGGT: `(3.764 + 2.236) / 2 = 3.000`
- Spann3R: `(2.028 + 2.202) / 2 = 2.115`
- UniDWM: `(1.878 + 1.575) / 2 = 1.7265`, rounded to `1.727`

The paper's textual claim in `sec/4_exp.tex` line 104 that UniDWM reduces Overall versus VGGT by `42.4%` also checks: `(3.000 - 1.727) / 3.000 = 42.43%`.

This supports the arithmetic consistency of the 4D reconstruction table, but not its empirical reproducibility because predictions, evaluation scripts, data split manifest, and code are absent.

### 2. The reported planning table supports the direction of the headline label-free planning claim, but not independent empirical confidence.

The main NAVSIM planning table in `sec/3_method.tex` lines 179-205 reports UniDWM(DINOv3-B) at `PDMS=90.6`, above the listed label-free baselines: Epona `86.2`, DINOv3(B) `85.4`, World4Drive `85.1`, LAW `83.8`, Ego-MLP `65.6`. Manual differences are:

- UniDWM(DINOv3-B) vs Epona: `+4.4` PDMS
- UniDWM(DINOv3-B) vs raw DINOv3(B): `+5.2` PDMS
- UniDWM(DINOv3-B) vs World4Drive: `+5.5` PDMS

So the table values, if trusted, do support "best among listed label-free methods." However, no variance, seed count, confidence intervals, checkpoint, script, or evaluation logs are provided. The claim is therefore not independently reproducible from the submitted artifacts.

### 3. The ablation arithmetic partly supports multifaceted supervision, but reveals an ambiguity around the headline DCAE number.

The DCAE ablation table in `sec/4_exp.tex` lines 63-82 reports:

- Ego-only baseline: `78.5` PDMS
- + Appearance: `81.3`, gain `+2.8`
- + Geometry only: `80.0`, gain `+1.5`
- + Dynamic generation only: `80.9`, gain `+2.4`
- + Geometry + Dynamic: `81.6`, gain `+3.1`
- + Appearance + Geometry + Dynamic: `82.4`, gain `+3.9`

These gains match the prose in `sec/4_exp.tex` line 111, except that the full model's total gain of `+3.9` is not foregrounded there. This is positive internal evidence that multiple reconstruction/generation heads are associated with higher planning scores in the reported DCAE setting.

The concerning inconsistency is that the main planning table reports `UniDWM (DCAE)` as `PDMS=84.9` in `sec/3_method.tex` line 201. That exact value appears in the GRPO table as `UniDWM w/ GRPO`, not as the plain DCAE full model, in `sec/4_exp.tex` lines 93-96. The non-GRPO full DCAE ablation model is `82.4`.

This matters because the headline table does not explicitly label `UniDWM (DCAE)` as GRPO-finetuned. If the DCAE entry includes GRPO while the ablation and method discussion distinguish non-GRPO UniDWM from UniDWM w/ GRPO, the reported headline comparison is under-specified. I cannot determine from the provided source whether the DINOv3-B headline value `90.6` is also GRPO-finetuned, plain supervised, or produced under a different recipe.

### 4. The InfoVAE/VAE reconstruction is mathematically plausible but weak as "theoretical guidance."

The VAE-style ELBO derivation in `main.tex` lines 144-256 is a standard conditional-latent Jensen lower bound under a strong conditional independence assumption:

```text
p_theta(x,z) = p(z) prod_j p_theta(x^(j) | z)
```

The algebra from the marginal likelihood to the multi-observation ELBO checks under that assumed factorization.

The InfoVAE-style objective in `main.tex` lines 258-400 decomposes the expected KL into mutual information plus aggregated posterior KL, then sets `alpha=1`, removing the mutual-information penalty and retaining only a divergence between aggregated posterior and prior. This is consistent as a formal objective template, but it does not by itself validate the concrete implementation or explain why the selected reconstruction/generation heads should be sufficient for planning. The practical objective is implemented as negative reconstruction/generation losses plus SIGReg per `sec/3_method.tex` lines 74-85, yet the artifacts do not provide the actual SIGReg implementation, latent sampling procedure, or loss weighting beyond `lambda=2e-4`.

Thus I can reconstruct the algebraic framing, but I do not view it as independently validating the central empirical claim.

### 5. The smoothness appendix is consistent arithmetically but not reproducible.

The smoothness table in `main.tex` lines 407-432 reports:

- kNN distance: baseline `130.578`, UniDWM `117.238`, a `10.22%` reduction.
- PCA ratio: baseline `0.554`, UniDWM `0.622`, an absolute increase of `0.068`.
- Laplacian smoothness: baseline `328,596`, UniDWM `263,981`, a `19.66%` reduction.

These deltas align with the prose in `main.tex` line 418. However, the analysis is not reproducible because it lacks the split manifest, latent extraction code, exact representation tensor choice, PCA dimensionality, kNN metric, graph weighting, normalization details beyond feature-wise standardization, and random/approximate nearest-neighbor settings.

## Blockers and Why They Matter

1. The claimed public code repository is unavailable. The abstract states that code will be available at `https://github.com/Say2L/UniDWM`, but the clone attempt failed with public 404/credential prompt. Without code, I cannot run training, inference, NAVSIM evaluation, SIGReg, GRPO finetuning, or 4D reconstruction metrics.

2. No checkpoints or prediction artifacts are included. Even if full training is too expensive, reproducibility would require at least checkpoints or saved predictions for NAVSIM `navtest` and the reconstruction benchmark.

3. The training recipe is incomplete for reimplementation. The paper gives high-level values in `sec/4_exp.tex` line 20: 50 epochs per stage, batch size 64, AdamW, learning rate `1e-4`, weight decay `5e-2`, image size `224x384`, `lambda=2e-4`, and DiT sampling steps 100/5. It does not specify seeds, hardware, schedule, warmup, augmentations, exact NAVSIM version, scene filtering, train/validation/test manifests, data loader details, trajectory decoder target parameterization, GRPO settings, or evaluation command.

4. Baseline comparability cannot be checked. Some table entries are cited from prior work, some are marked `*` as obtained by the authors, and the main UniDWM(DCAE) number appears to match the GRPO-finetuned value rather than the plain ablation full model. This prevents a clean apples-to-apples reproduction of the headline planning comparison.

5. Generation quality is only qualitative in the final text. A commented-out table in `sec/4_exp.tex` lines 23-36 contains possible FID/FVD/LPIPS numbers, but it is not part of the paper's active results. The visible 4D generation claim is therefore not quantitatively reproducible from the supplied text.

## Match / Partial Match / Mismatch / Blocked

Outcome: **Partial match on internal arithmetic; blocked for empirical reproduction.**

What matches:

- 4D reconstruction Overall values and the `42.4%` reduction claim are arithmetically correct.
- Planning table values support the claim that UniDWM(DINOv3-B) is best among the listed label-free methods, assuming the table is trusted.
- Ablation deltas for appearance, geometry, and dynamic generation are arithmetically correct.
- The VAE-style derivation is algebraically valid under the paper's assumed conditional independence model.

What does not fully match or remains unresolved:

- The main `UniDWM (DCAE)` planning entry `84.9` matches `UniDWM w/ GRPO`, not the non-GRPO full ablation `82.4`. This is a material reporting ambiguity.
- The InfoVAE framing is a plausible objective reinterpretation, not an empirical reproduction of learned world-model behavior.
- No executable path exists from the submitted artifacts to recover the NAVSIM planning, reconstruction, generation, or smoothness numbers.

## Agreement With Reproducer A

No Reproducer A report was present in `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/` at the time of this pass, so I cannot compare conclusions directly. My independent outcome should be compared against Reproducer A once their report is available. If Reproducer A used attempted code execution, this report provides a distinct source/table/derivation route.

## Final Synthesis and Score Impact

The paper's reported tables are not obviously arithmetically fabricated: the reconstruction, ablation, and smoothness calculations I could recompute are internally coherent. The strongest manually validated evidence is that the reported ablation table consistently associates additional appearance, geometry, and dynamic-generation supervision with higher DCAE planning PDMS.

However, the central empirical claim is **not independently reproducible** from the available artifacts. The implementation repository is unavailable, no checkpoints or predictions are supplied, key training/evaluation details are missing, and one headline planning number is ambiguous with respect to GRPO finetuning. Under a reproducibility-first standard, this should materially reduce confidence in the paper. I would treat the claim as **partially supported by reported internal consistency but weakly reproducible overall** until code, checkpoints, split manifests, and evaluation scripts are provided.
