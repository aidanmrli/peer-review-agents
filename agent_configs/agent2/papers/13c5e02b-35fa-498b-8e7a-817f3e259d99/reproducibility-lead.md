# Reproducibility Lead Report: UniDWM

Paper ID: `13c5e02b-35fa-498b-8e7a-817f3e259d99`
Title: "UniDWM: Towards a Unified Driving World Model via Multifaceted Representation Learning"
Assigned role: Reproducibility Lead
Date: 2026-04-24

## Task Scope

I coordinated the internal review of UniDWM under the agent2 reproducibility-first protocol. The central acceptance-relevant claim tested was that UniDWM learns a multifaceted latent driving-world representation which substantially improves NAVSIM trajectory planning while also supporting 4D reconstruction and future generation.

Minimum reproduction targets were set before consolidation:

- Recover or independently verify the headline NAVSIM planning result, especially UniDWM with DINOv3-B reporting `PDMS = 90.6` on `navtest`.
- Verify the 4D reconstruction table claim that UniDWM reaches `Overall = 1.727` and improves over VGGT by `42.4%`.
- Inspect whether the advertised GitHub artifact supports code-level reproduction of training, evaluation, ablations, GRPO finetuning, and smoothness analysis.
- Check whether the VAE/InfoVAE framing and loss definitions are technically sound.
- Check whether the novelty framing is supported relative to prior driving world model and latent representation work cited by the paper.

Tolerance: for numerical table checks, exact arithmetic consistency or standard rounding was required. For empirical reproduction, at least two independent roles needed to recover a reported result, model behavior, metric computation, or executable sub-pipeline from official artifacts. Merely verifying arithmetic in static tables was not counted as reproducing the empirical claim.

## Evidence Examined

Official Koala artifacts:

- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/paper.pdf`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/source.tar.gz`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/0_abstract.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/1_intro.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/2_related.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/3_method.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/4_exp.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.bib`

Advertised implementation artifact:

- `https://github.com/Say2L/UniDWM`

Role reports:

- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

## Role Findings

### Independent Reproducer A

Reproducer A started from the paper text and official artifacts. The central NAVSIM planning claim could not be reproduced because the artifact archive contains only LaTeX, figures, bibliography, and style files. No runnable code, checkpoints, data subset, preprocessing, split manifests, evaluation scripts, seeds, or logs were present. The advertised GitHub repo returned HTTP 404 / credential prompt under public access. Reproducer A verified only static table arithmetic, including the `42.4%` reconstruction reduction and selected planning/ablation differences. Outcome: blocked for central empirical reproduction.

### Independent Reproducer B

Reproducer B used an independent source/table/derivation route. This pass verified that the 4D reconstruction `Overall` values are arithmetically consistent with averaging Accuracy and Completeness, that the ablation deltas are internally consistent, and that the reported planning table supports the direction of the headline label-free planning claim if the table is trusted. Reproducer B also found a reporting ambiguity: the main table reports `UniDWM (DCAE)` at `PDMS = 84.9`, which matches the `UniDWM w/ GRPO` row in the GRPO table, whereas the non-GRPO full DCAE ablation model is `82.4`. Outcome: partial arithmetic validation but blocked for empirical reproduction.

### Implementation Auditor

The implementation auditor confirmed that the listed repository `https://github.com/Say2L/UniDWM` is not publicly reachable: `curl` and GitHub API probes returned `404`, and `git ls-remote` failed without credentials. Consequently, no paper-to-code comparison was possible. Missing artifacts include model definitions, training scripts, NAVSIM preprocessing, PDMS evaluator invocation, ablation configs, GRPO finetuning setup, smoothness scripts, checkpoints, prediction dumps, logs, dependency pins, seeds, and environment instructions. Outcome: high-severity artifact failure for an empirical paper.

### Correctness Specialist

The correctness specialist found that the reported arithmetic is mostly internally consistent, including the `42.4%` 4D reconstruction improvement. However, the paper has major technical weaknesses:

- The InfoVAE-style objective is described as an ELBO even though reweighting/removing the mutual-information term and replacing KL with a generic divergence does not preserve a lower-bound guarantee without additional assumptions.
- The actual architecture is specified mostly as deterministic encoders/decoders, not as a probabilistic VAE with defined posterior parameters, reparameterization, and likelihood families.
- The uncertainty-weighted geometry loss multiplies residuals by `Sigma` and subtracts `a log Sigma`, which gives larger residuals smaller optimal `Sigma` and is unbounded for zero residual if implemented literally.
- The diffusion velocity objective lacks a defined forward noising/interpolation process and sign convention.
- The 4D generation claim is qualitative; the quantitative generation table is commented out.
- The ablation table supports component utility but not the stronger "mutually reinforcing" conclusion.

Outcome: major correctness downgrade on theoretical and generative claims.

### Literature Specialist

The literature specialist found that UniDWM is a competent synthesis of recent driving world models, latent representation learning, visual geometry reconstruction, and diffusion generation, but the novelty framing is overstated. The paper's own bibliography includes HERMES, World4Drive, LAW, DrivingWorld, Epona, DriveX, and related generative driving systems. The defensible contribution is the particular multifaceted supervision recipe and NAVSIM planning transfer, not a fundamentally new unified driving world model or VAE-theoretic formulation. The claim of learning "solely from visual observations" is also too strong because the method uses ego status and LiDAR-projected point/depth supervision. Outcome: moderate novelty downgrade.

## Commands and Checks

Repository reachability:

```bash
curl -fsSL -o /dev/null -w "%{http_code}\n" https://github.com/Say2L/UniDWM
git ls-remote https://github.com/Say2L/UniDWM.git
```

Observed:

```text
404
fatal: could not read Username for 'https://github.com': No such device or address
```

Artifact inventory:

```bash
find papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts -maxdepth 2 -type f | sort
```

Observed: LaTeX source, bibliography/style files, PDF figures, paper PDF, and source tarball. No executable implementation or evaluation artifact.

Arithmetic check:

```bash
python - <<'PY'
checks = {
    'UniDWM DINOv3-B PDMS vs Epona': (90.6 - 86.2, (90.6 - 86.2)/86.2*100),
    'UniDWM DINOv3-B PDMS vs World4Drive': (90.6 - 85.1, (90.6 - 85.1)/85.1*100),
    'UniDWM DINOv3-B PDMS vs DINOv3 raw': (90.6 - 85.4, (90.6 - 85.4)/85.4*100),
    '4D overall reduction vs VGGT': (3.000 - 1.727, (3.000 - 1.727)/3.000*100),
    'Ablation appearance gain': (81.3 - 78.5, None),
    'Ablation geometry gain': (80.0 - 78.5, None),
    'Ablation dynamic gain': (80.9 - 78.5, None),
}
for name, (delta, pct) in checks.items():
    print(name, delta, pct)
PY
```

Observed:

- UniDWM DINOv3-B exceeds Epona by `4.4` PDMS, World4Drive by `5.5`, and raw DINOv3 by `5.2`, if the table is trusted.
- The 4D Overall reduction versus VGGT is `42.43%`, matching the paper's `42.4%`.
- The ablation gains of `+2.8`, `+1.5`, and `+2.4` match the reported table.

## Reproducibility Outcome

Central empirical claim: weak reproducibility.

Both independent reproducers failed to reproduce the central NAVSIM planning result, reconstruction metric, generation behavior, ablation, GRPO result, or smoothness analysis from official artifacts. Both could verify only arithmetic consistency in static tables. The implementation auditor independently established that the advertised GitHub repository is not publicly accessible. Under this protocol, arithmetic consistency is not a reproduction of the scientific claim.

Strongly supported items:

- The paper's reported table arithmetic is mostly internally consistent.
- The method is understandable at a high level from the LaTeX source.

Partially supported items:

- The reported ablation pattern is internally coherent, but not independently executable.
- The VAE-style algebra for the initial multi-observation ELBO is valid under the stated conditional-independence assumption, but the final InfoVAE-style training objective is not established as an ELBO.

Unsupported or unreproduced central items:

- UniDWM (DINOv3-B) `PDMS = 90.6` on NAVSIM `navtest`.
- UniDWM reconstruction `Overall = 1.727`.
- 4D generation effectiveness.
- GRPO finetuning result `PDMS = 84.9`.
- Smoothness metrics and their link to robustness/generalization.
- Paper-code match for the described method.

## Decision Impact

The paper has a plausible systems idea and, if the tables are valid, strong NAVSIM performance. However, the evidence remains paper-only. For an empirical autonomous-driving world-model submission, unavailable code/checkpoints/evaluation artifacts and under-specified probability/diffusion details materially reduce confidence. I would not treat the headline results as independently established.

Recommended score range under agent2's reproducibility-first rubric: `3.5` to `4.5` unless the missing artifacts become public and allow independent verification. This is a weak-reject range driven by unreproduced central claims, overstated theoretical framing, and moderate novelty concerns.

## Remaining Uncertainty

The results may be correct; the review does not prove otherwise. The uncertainty is that reviewers cannot currently inspect or rerun the system. Because the paper's acceptance case depends on exact data processing, training, model implementation, and NAVSIM evaluation, that uncertainty is decision-relevant rather than incidental.
