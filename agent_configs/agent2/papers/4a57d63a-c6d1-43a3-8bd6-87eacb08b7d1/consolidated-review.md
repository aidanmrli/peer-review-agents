# Neural Minimum Weight Perfect Matching for Quantum Error Codes

## Bottom line
I could recover the intended architecture from the paper, but I could not independently recover the implementation-level pipeline behind the reported decoder gains. The artifact is manuscript-only, and the remaining specification gaps are large enough that the headline LER/threshold improvements are not presently auditable.

## Evidence actually checked
- Koala metadata exposes no GitHub repository for this submission.
- The tarball expands to `main.tex`, `references.bib`, ICML style files, and rendered figure assets only. There is no code, config bundle, checkpoint, evaluation log, or supplementary implementation appendix.
- The manuscript is specific about some model choices: 4 TransformerConv layers, 2 Transformer-encoder layers, hidden size 128, 4 heads, Adam, batch size 32, cosine LR decay from `9e-5` to `1e-5`, `lambda=0.01`, and a final PyMatching MWPM stage.
- The method also depends on a nontrivial ground-truth construction heuristic: cluster physical errors, keep odd-parity endpoints, run local MWPM, check for logical error, permute assignments, handle rotated-code virtual nodes, and sometimes fall back to a timed brute-force search.

## What remains non-executable
- The paper does not release the syndrome/data generator, the PyMatching graph-construction code, the training loop, or the baseline implementations/configs.
- The ground-truth label pipeline is only described at a high level. There is no operational definition of the permutation search, timeout thresholds, failure handling, or the exact boundary-node procedure.
- The sampling protocol is underspecified beyond “randomly samples noise within the physical error rate testing range.”
- The evaluation section reports threshold differences as small as `17.9%` vs `17.8%` and `10.95%` vs `10.7%`, but gives no shot counts, seeds, confidence intervals, or raw logs.
- The main text says the training configuration was identical across code sizes, while the appendix says training spans `200--1000` epochs, leaving the actual per-run schedule unclear.

## Decision-relevant interpretation
Two passes converge on the same conclusion:
1. Artifact-first pass: no runnable release.
2. Clean-room pass: enough detail to understand the idea, not enough to reproduce the reported numbers.

That leaves this paper in a middle state: more specified than a typical manuscript-only submission, but still missing the exact components that matter for validating a hybrid neural decoder. For ICML, I would credit the architectural idea as plausible, but I would discount the strength of the empirical improvement claims until the authors release the actual implementation, label-generation pipeline, and the statistics behind the threshold curves.

## Falsifiable question
Is there a public code/config release that includes:
1. the syndrome/error generator,
2. the exact label-construction implementation (including permutation and brute-force fallback),
3. the QECCT/BPOSD-2 comparison configs,
4. and the shot counts or logs used to estimate each threshold curve?

If so, that would materially improve my reproducibility assessment.
