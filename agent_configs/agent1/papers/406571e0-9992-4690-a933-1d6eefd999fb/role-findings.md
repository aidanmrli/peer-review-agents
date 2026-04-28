# VEQ Reproducibility Findings

## Central claim and reproduction target
VEQ claims modality-aware MoE VLM PTQ gains from two components, VEQ-ME and VEQ-MA, with reproducible benchmark improvements under W3A16/W4A16.

## Paper and artifact evidence checked
- Koala tarball contents: `arxiv-main.tex`, figures, bib, styles; no runnable code.
- Linked repo README at `guangshuoqin/VEQ`.
- Source passages in `arxiv-main.tex` for abstract, implementation details, component ablations, and sensitivity analysis.

## Smallest meaningful check actually run
- Confirmed the public GitHub repo resolves but README still says `Our code will be available at https://github.com/qsstcl/VEQ`, marks `Complete this repository` and `Release the code` unchecked, and points supplementary material to `https://github.com/`.
- Confirmed from `arxiv-main.tex` that Section 4.3 sets hyperparameters to `default optimal values`, while Section 4.4 says those values come from a grid search over 64 randomly extracted MMMU validation samples. The manuscript does not list the chosen `gamma`, `beta`, or `lambda` values, nor the sampled IDs.

## Implementation or correctness risks
- Reproducing Tables 2-3 and the sensitivity figures requires the hidden defaults and the hidden 64-sample subset.
- The missing code is not the only blocker: the paper text itself does not uniquely identify the calibration path behind the reported defaults.
- This makes the load-bearing modality-balance parameters effectively un-auditable from the current public release.

## Novelty/framing context
- Other reviewers already established the unified-framework framing gap and the README-only repository state.
- My added point is narrower: even if code were later uploaded, the current paper does not expose the exact hyperparameter/calibration state used for the reported VEQ-ME/VEQ-MA numbers.

## Decision impact
- Supports a conservative reproducibility score.
- Would change with a concrete release of the exact `gamma/beta/lambda` defaults, the MMMU sample manifest used for the grid search, and runnable quantize/evaluate commands.
