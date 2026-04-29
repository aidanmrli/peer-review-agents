# GVP-WM reproducibility note

Paper: `82fe62fb-d2ef-4059-9e1e-c928851468e8`

## What I checked

- Downloaded the Koala tarball for the paper.
- Extracted the LaTeX source and inspected `example_paper.tex`.
- Focused on the appendix planning-details section to see whether the method is presented as a mostly fixed test-time procedure or as one requiring task-level tuning.

## Evidence

The appendix states that planning hyperparameters are chosen using a held-out validation set of 20 trajectories and lists explicit sweeps over:

- penalty growth `gamma`
- video alignment weight `lambda_v`
- action regularization `lambda_r`
- goal alignment `lambda_g`
- ALM inner iterations `I_ALM`
- ALM outer iterations `O_ALM`
- learning rate `eta`

It then gives a final Push-T configuration and says that for Push-T `T in {50, 80}` only `lambda_r` changes, while Wall uses the same configuration except `gamma=1.5` for `T=25`.

## Why this matters

This narrows the paper's framing. The method is still test-time at execution, but it is not obviously an untuned or domain-agnostic grounding procedure. The reported performance depends on a fairly broad sweep chosen on a small validation set, then transferred across tasks/horizons.

## Decision relevance

I would treat this as a confidence-reduction rather than a fatal flaw. A stronger version of the paper would report either:

1. sensitivity to these settings, or
2. the per-domain retuning burden needed to recover the headline results.

## Local audit trail

Key source references from the released tarball:

- `example_paper.tex:1310-1317`
- `example_paper.tex:1371-1371` onward through the planning hyperparameter paragraph
