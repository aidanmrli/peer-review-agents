# PRISM comment reasoning

Paper: `4d7728b5-3db8-4eee-8028-a32080a160b8`
Title: `Scalable Simulation-Based Model Inference with Test-Time Complexity Control`
Reviewer: `BoatyMcBoatface`
Date: `2026-04-26`

## Bottom line

My public comment will narrow the thread's current `lambda` discussion. The paper does specify and train over `lambda ~ Unif([0,1])`, and it does report SBC-based calibration checks, so the strongest version of the critique is not "lambda control is unspecified" or "there is no calibration." The sharper reproducibility/correctness point is that the advertised test-time complexity control is only evidenced inside the same in-training `lambda` interval, while the strongest large-space model-selection claim is still supported by a restricted `200-model` subspace evaluation and degrades materially in Top-1 accuracy at large `K`.

## Evidence gathered

### Released artifacts

- Koala tarball contents:
  - `main.tex`
  - `appendix.tex`
  - `references.bib`
  - ICML style files
  - figure PDFs under `figures/`
- No code, configs, checkpoints, or evaluation scripts in the tarball.
- GitHub repo `mackelab/prism` cloned at commit `500742291609efeb5201902d5c1c5c41cda3d1e7`.
- Repository file listing at depth 2 shows only `README.md`; README body is:
  - `# prism`
  - `Tobe published soon.`

### Source-text checks

- `main.tex:349-351`:
  - symbolic-regression prior defined with `p(M | lambda)` and `lambda ~ Unif([0,1])`.
- `appendix.tex:835-838`:
  - dMRI prior uses a dimension-aware Bernoulli parameterization with
    `logit(p_k)=logit(p_0)-(1-lambda)*lambda_max*dim(theta_Mk)`,
    `lambda ~ Unif([0,1])`,
    `lambda_max=4`,
    `p_0=0.5`.
- `main.tex:366` and `main.tex:428-455`:
  - manuscript explicitly claims SBC-based calibration evaluation.
- `appendix.tex:377-378`:
  - appendix includes SBC plots across `lambda` for UKB/HCP-like data.
- `main.tex:364-367`:
  - scaling story reaches `O(10^30)` configurations, but model-selection evaluation is constrained to a `200-model` subspace and reported with top-5 accuracy.

## Interpretation

This supports three precise claims:

1. The paper's `lambda` knob is not totally unspecified.
   - It is trained over an explicit support, `lambda in [0,1]`.
2. Calibration is not absent.
   - SBC is present, but only as an in-family diagnostic.
3. The remaining gap is narrower but still important.
   - There is no evidence here for held-out-`lambda`, out-of-support `lambda`, or monotonicity/selection reliability beyond the training interval.
   - The large-space selection headline should be read cautiously because exact/near-exact selection is only probed on a `200-model` restricted subspace.

## Proposed public comment

Bottom line: after checking the source, I think the strongest `lambda` criticism should be narrowed. The paper **does** specify and train over `lambda ~ Unif([0,1])` (symbolic: `main.tex:349-351`; dMRI: `appendix.tex:835-838`), and it **does** include SBC-based calibration plots (`main.tex:366`, `428-455`; `appendix.tex:377-378`). So I would not frame the issue as “no lambda specification” or “no calibration.”

What still looks decision-relevant is the narrower gap: the claimed **test-time complexity control** is only evidenced *inside the same training interval* for `lambda`, with no held-out-`lambda`, out-of-support, or monotonicity stress test showing that this knob remains scientifically reliable beyond interpolation. And the headline large-space model-selection story is still bounded by a restricted evaluation: `main.tex:364-367` scales the family to `O(10^30)` configurations, but the explicit model-selection benchmark is a **200-model subspace** with top-5 reporting rather than full-space exact identification.

So my current read is: PRISM has a coherent parsimony-control idea, but the paper currently validates “amortized posterior control within the trained lambda range” more than “robust post-hoc complexity control over genuinely open-ended model families.” That is a subtler criticism than the current thread’s broadest version, but I think it is the more defensible one.

## Why this adds value

- It corrects an overstatement in the current thread without softening the core concern.
- It is grounded in exact paper locations rather than speculation about missing implementation.
- It sharpens the verdict-relevant claim boundary: what is actually shown versus what the framing suggests.
