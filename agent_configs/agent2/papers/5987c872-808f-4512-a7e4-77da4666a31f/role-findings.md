# Central claim and reproduction target
Central claim checked: the paper reports that SCD recovers causal graphs more accurately than DYNOTEARS, PCMCI, SCOTCH, and Neural-GC on synthetic dynamical systems with partial mechanistic priors. My targeted check was narrower: whether the comparison protocol uses a consistent edge-selection procedure across methods, since all reported SHD/TPR/FDR scores depend on thresholding estimated graphs.

## Paper and artifact evidence checked
- Paper source: `/tmp/physicscausal/PICD_preprint.tex`
- Main experimental setup: Section `Data Generation and experimental setup`, especially the thresholding paragraph around lines 640-642.
- Representative result tables: `tab:picd_d5_compact`, `tab:dag_stable`, and nearby text.

## Reproducibility result from the smallest meaningful check
I could verify that the benchmark comparison relies on materially different post-hoc threshold choices across methods, with no common calibration rule stated.
- The paper says, “After obtaining the estimated graphs, we apply thresholding to all methods.”
- For SCD, it reports that “a threshold in the range `[0.20, 0.25]` works well across all settings.”
- For DYNOTEARS, the authors explicitly override the implementation’s recommended thresholds because they produced empty graphs, switching to `0.03` for contemporaneous edges and `0.01` for lagged edges.
- SCOTCH is thresholded at `0.5`; all other baselines use “recommended settings.”

## Implementation or correctness risks
- Evaluation-fairness risk: SHD/TPR/FDR can move sharply with threshold. If thresholds are chosen by what “works well” per method rather than by a shared validation rule, part of the reported gain may come from threshold calibration rather than better causal recovery.
- Asymmetric tuning risk: DYNOTEARS is not run at its default decision rule, but SCD also appears to use a hand-chosen threshold range rather than a fixed pre-registered cutoff.
- Reporting gap: I did not find PR curves, threshold-sensitivity plots, or a validation-set protocol that would let a reviewer separate estimator quality from threshold selection.

## Novelty or framing context
Other reviewers focused on the drift-vs-diffusion modeling assumption. My contribution is orthogonal: even accepting that modeling choice, the empirical comparison should still be robust to a common edge-selection protocol.

## Decision impact
This does not by itself refute SCD, but it weakens how strongly I can trust the magnitude of the reported benchmark gap. A convincing revision would report threshold-sensitivity curves or a shared validation-based calibration rule for every method.
