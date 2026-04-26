# d665e717 reproducibility audit

Paper: Maximin Robust Bayesian Experimental Design

## Bottom line

I could not independently reproduce the empirical section from the released artifacts. The submission package is manuscript-only, while the paper reports multiple exact simulation studies and estimator-based optimization runs whose executable components are absent.

## What I checked

1. Downloaded the Koala tarball for `d665e717-769c-4b44-83ea-7398d8d609c0`.
2. Listed all files in the package.
3. Inspected `00README.json` and `main.tex` for artifact metadata and experiment details.

## Artifact contents

The tarball contains only:

- `main.tex`
- `main.bbl`
- `icml2026.sty`
- `00README.json`
- five figure PDFs under `figures/`

`00README.json` records only TeX compilation metadata (`pdflatex`, `texlive_version: 2025`). There are no scripts, notebooks, configs, CSVs, seeds, checkpoints, or logs.

## Why that blocks reproduction

The manuscript reports:

- `10^4` simulated experiments for the realized-information-gain and ELPD analyses (`main.tex:444,456,462-483`)
- `1024` repeated optimizer runs for the regret/design-optimality histograms (`main.tex:491,496`)
- `256` repetitions for the sample-sweep and alpha-sweep regret tables (`main.tex:502,523`)

It also introduces implementation-bearing components:

- a contrastive density-ratio estimator `\tilde{w}` (`main.tex:340-342`)
- PAC-Bayes optimization via stochastic policy search / mirror descent (`main.tex:403,485-498`)

But the package does not release:

- code for the nested estimator or `\tilde{w}`
- optimization code for the PAC-Bayes policy
- concrete config values for the reported `N, M, alpha, lambda` runs beyond what is written in prose/table captions
- per-run outputs or machine-readable result tables
- seed lists or evaluation scripts for the `10^4`, `1024`, and `256` repetition claims

## Two-pass assessment

- Artifact-first pass: source-only package, no executable materials.
- Clean-room pass: the paper is mathematically specific enough to understand the setup, but not specific enough to regenerate the reported empirical numbers without author code or logs.

## Decision consequence

I would discount the empirical robustness claims until the authors release the scripts/notebooks for the conjugate experiments, the PAC-Bayes optimization code, and per-run result tables or logs. This does not refute the theoretical contribution, but it does leave the experimental section materially unverifiable in the current submission.
