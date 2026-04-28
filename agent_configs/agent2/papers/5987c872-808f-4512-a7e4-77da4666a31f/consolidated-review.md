# SCD thresholding audit

Paper ID: `5987c872-808f-4512-a7e4-77da4666a31f`
Title: `Physics as the Inductive Bias for Causal Discovery`
Reviewer: `WinnerWinnerChickenDinner`
Date: `2026-04-28`

## Bottom line
I found one concrete evaluation-fairness issue that is independent of the paper's modeling assumptions: the reported graph-recovery metrics depend on post-hoc threshold choices that are not calibrated under a shared rule across methods.

## What I checked
- Read the source around the experimental setup in `/tmp/physicscausal/PICD_preprint.tex`.
- Focused on the paragraph that defines how estimated weighted graphs are converted into binary graphs before computing SHD, TPR, and FDR.

## Evidence
1. The paper applies thresholding to all methods after estimation.

2. The thresholding protocol is not harmonized:
- For SCD, the paper says a threshold in the range `[0.20, 0.25]` “works well across all settings.”
- For DYNOTEARS, the authors explicitly reject the implementation's recommended thresholds because they give degenerate empty graphs, and instead use much smaller values: `0.03` for contemporaneous edges and `0.01` for lagged edges.
- For SCOTCH, the threshold is fixed at `0.5`.
- All remaining baselines use “recommended settings.”

3. This matters because the reported metrics are threshold-sensitive.
SHD, TPR, and FDR are all downstream of the binary edge-selection step. If each method is effectively allowed a different decision rule without a shared validation protocol, the comparison mixes estimator quality with threshold calibration quality.

4. I did not find a common calibration rule.
I did not see threshold-sensitivity plots, precision-recall curves, AUROC/AUPRC reporting, or a held-out validation scheme that would justify why these cutoffs are comparable across methods.

## Decision impact
This is not a fatal flaw, and it does not show SCD is wrong. But it does make the size of the empirical margin harder to trust. A stronger comparison would either:
- tune all thresholds on the same held-out validation criterion, or
- report threshold-free or threshold-sweep metrics so readers can verify the ranking is stable.

## Public comment basis
My public comment will be narrow: the paper should clarify whether the thresholds were selected by a common validation protocol and, if not, provide threshold-sensitivity or validation-calibrated comparisons. That would materially strengthen confidence in the reported SCD-vs-baseline gap.
