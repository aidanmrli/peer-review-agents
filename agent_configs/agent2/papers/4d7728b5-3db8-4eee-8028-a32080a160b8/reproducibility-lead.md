# Reproducibility Lead Report

Paper ID: `4d7728b5-3db8-4eee-8028-a32080a160b8`

Title: "Scalable Simulation-Based Model Inference with Test-Time Complexity Control"

Assigned role: Reproducibility Lead for agent2

Date: 2026-04-24

## Task Scope

I coordinated the internal review team and synthesized whether the core claims were independently reproducible from the paper, source archive, and official linked artifacts.

Central claims tested:

1. PRISM infers a joint posterior over discrete model structure and continuous parameters.
2. PRISM enables test-time model-complexity control through a tunable model prior.
3. PRISM scales to very large symbolic-regression model spaces with calibrated useful predictions.
4. PRISM supports dMRI model selection and parameter inference, including evidence alignment and downstream tractography improvements.

Minimum reproduction target:

- At least one executable reproduction of a reported symbolic or dMRI result, or a smaller meaningful unit such as simulator generation, posterior sampling, rRMSE/SBC/KSD computation, evidence-estimator computation, or inference from a released checkpoint.
- Two independent reproducers were asked to use different routes and not copy each other.

## Evidence Examined

- Koala paper metadata and existing discussion thread.
- Submitted PDF at `artifacts/paper.pdf`.
- LaTeX source archive under `artifacts/source`, including `main.tex`, `appendix.tex`, figures, and bibliography.
- Official linked GitHub repository cloned at `artifacts/prism`, commit `500742291609efeb5201902d5c1c5c41cda3d1e7`.
- Internal role reports:
  - `independent-reproducer-a.md`
  - `independent-reproducer-b.md`
  - `implementation-auditor.md`
  - `correctness-specialist.md`
  - `literature-specialist.md`

Commands and checks included:

```bash
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism ls-tree -r --name-only HEAD
sed -n '1,200p' papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism/README.md
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism show --stat --format=fuller --no-renames HEAD
rg -n "Code to reproduce|billions|rRMSE|SBC|KSD|R\\^2|tractography|H100|72 hours|model selection" papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source
```

## Role Findings

### Independent Reproducer A

Outcome: blocked for empirical reproduction.

Reproducer A verified only simple arithmetic and prior-level behavior: the symbolic Bernoulli prior gives expected active components `K * lambda`, and the paper's stated batch-size/update-count arithmetic is consistent with billions of simulated examples. A could not run PRISM, generate the authors' simulations, reproduce rRMSE/SBC/KSD, evaluate dMRI evidence alignment, or reproduce tractography metrics because the official repository contains no implementation.

Reproducibility classification: weak for central empirical claims; partial only for prior arithmetic.

### Independent Reproducer B

Outcome: blocked for empirical reproduction.

Reproducer B independently confirmed the model-space arithmetic and prior-level test-time complexity control, but emphasized that the headline learned-generalization claim is not a derivation. At `K=50` and above, only a tiny fraction of the model space can be sampled during training, making the missing code/checkpoints especially important. B also noted that symbolic top-5 classification is evaluated on a 200-model subspace, not the full combinatorial space.

Reproducibility classification: weak for central empirical claims; partial only for prior arithmetic.

### Implementation Auditor

Outcome: major artifact failure.

The official `https://github.com/mackelab/prism` clone contains only `README.md`, whose substantive content is:

```text
# prism

Tobe published soon.
```

No implementation, environment files, Hydra configs, simulator code, training scripts, evaluation scripts, checkpoints, raw metric files, generated datasets, seeds, figure-generation code, or downstream neuroimaging pipeline artifacts were present. This directly contradicts `main.tex` lines 641-642, which state that code to reproduce results is available at the repository URL.

Reproducibility classification: weak.

### Correctness Specialist

Outcome: several major evidentiary concerns.

The strongest dMRI evidence-alignment and ESS claims require pointwise evaluation of the learned diffusion posterior density `q(theta | M, x)`, but the method describes an EDM-style diffusion sampler without specifying normalized density evaluation. The dMRI full-space "model discovery" procedure is only a 100-posterior-sample heuristic, not robust global optimization. The dMRI model-space/prior/table specification has internal inconsistencies around mutually exclusive noise indicators, reused component/noise IDs, and some prior units/constraints. The symbolic scaling evidence supports billion-scale predictive amortization more than reliable full-space posterior model identification.

Correctness classification: major limitations in support for exact evidence-based validation and robust dMRI model discovery.

### Literature Specialist

Outcome: novelty is plausible but narrower than broad framing.

PRISM is not a first proposal for joint model/parameter simulation-based inference: SBMI and amortized Bayesian model comparison cover important parts of the space. The credible contribution is a useful combination of SBMI-style model latents, all-in-one/Simformer-style flexible amortized inference, explicit test-time model-prior control, and larger symbolic/dMRI settings. The main literature gaps are under-discussion of TabPFN/in-context Bayesian inference, Distribution Transformers, and dMRI model-ranking context.

Literature classification: mildly negative to neutral; not a rediscovery, but novelty claims should be tightened.

## Synthesis

The internal team did not reproduce the core empirical claims. Both independent reproducers reached the same conclusion by separate routes: the simple prior-control arithmetic is valid, but the acceptance-critical trained-posterior claims cannot be checked from the official artifacts. The implementation auditor found the decisive blocker: the linked repository contains no code despite the paper's explicit software-availability statement.

The paper may contain a promising methodological idea. The symbolic prior and model-space arithmetic are coherent, and the literature pass suggests the combination is plausibly novel enough if framed as an extension rather than a first-of-kind method. But the evidence needed to trust the claimed performance is missing. In particular, the team could not independently reproduce:

- PRISM training or inference.
- Symbolic rRMSE/SBC/rKSD curves.
- Symbolic top-5 model-selection table values.
- dMRI evidence-alignment `R^2 = 0.97`.
- dMRI ESS/KSD/RMSE tables.
- Runtime claims on H100.
- Tractography correlation improvement over the SBI baseline.

Correctness concerns further limit the evidentiary force of the unreproduced dMRI claims: density evaluation for diffusion posterior ESS/evidence is not specified, full-space model discovery is a small-sample heuristic, and the extended dMRI model family is not specified cleanly enough to reconstruct.

## Reproduction Outcome

- Independent Reproducer A: blocked for central empirical claims; prior arithmetic checked.
- Independent Reproducer B: blocked for central empirical claims; prior/model-space arithmetic checked independently.
- More than one internal reproducer reproduced the core empirical claim: no.
- More than one internal reproducer validated a small non-central unit: yes, the prior-level lambda control and model-space arithmetic.

## Decision Impact

Substantial downgrade under agent2's reproducibility-first standard. The central claim should be treated as weakly reproducible, not strongly supported. A reasonable verdict range based on current evidence is weak reject to low weak accept depending on how much weight is assigned to the conceptual contribution versus artifact failure. My own current range is approximately 4.0-5.5, with the lower half favored unless missing artifacts appear before the verdict window.

## Remaining Uncertainty

The authors may have private code and checkpoints, but unavailable artifacts do not support independent review. The paper's reported results could be correct, but the current submission does not permit the internal team to verify them.
