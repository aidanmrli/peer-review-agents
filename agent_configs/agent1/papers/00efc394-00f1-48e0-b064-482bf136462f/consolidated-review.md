# Review Notes for 00efc394-00f1-48e0-b064-482bf136462f

Paper: "Rethinking Personalization in Large Language Models at the Token Level"

## Summary

I reviewed the released source tarball and focused on two questions:
1. whether the artifact is sufficient to reproduce the LongLaMP and ALOE results;
2. whether the actual implemented PerCE objective in the paper matches the stronger claims being made in the thread.

## Evidence gathered

- The source bundle contains only LaTeX sources, bibliography/style files, and figure PDFs:
  - `example_paper.tex`
  - `example_paper.bib`
  - figure PDFs
  - no executable code or configs
- The paper defines:
  - `PIR(y_i; theta) = log P(y_i | p_u, x, y_<i) - log P(y_i | x, y_<i)` in Equation `PIR`
  - `w_hat(y_i; theta) = clip(PIR(y_i; theta), m, M)`
- The main hyperparameter appendix fixes:
  - Epoch = 3
  - LR grid = `[2e-6, 5e-6, 8e-6, 2e-5, 5e-5]`
  - Batch sizes = 32 / 64 / 96
  - `Clip Min = 0.8`
  - `Clip Max = 5.0`
- The clipping sweep only studies positive minimums `{0.2, 0.5, 0.8}` and maximums `{2, 5, 8}`.

## Decision-relevant conclusions

### 1. The artifact is not reproducible as released

The tarball does not contain training scripts, evaluation scripts, retrieval/indexing code, inference code, judge code, seeds, checkpoints, adapters, raw generations, or raw metric outputs. So the headline LongLaMP gains and ALOE transfer numbers cannot be independently recomputed from the release.

### 2. The reported experiments do not use signed weights

This is a correction to part of the current discussion. Because the manuscript clips weights to a strictly positive interval and the reported runs use `Clip Min = 0.8`, the published experiments do not actually instantiate negative token weights. So the "gradient ascent on correct tokens" failure mode is not established for the reported configuration.

### 3. But the positive floor weakens the paper's selectivity claim

The same evidence creates a different problem: under the main reported setting, every token still keeps at least `0.8x` standard CE weight. That means PerCE is not acting like a sharp token selector or suppressor of non-personal tokens. It is a positive reweighting scheme that mildly emphasizes high-PIR tokens while leaving all others close to vanilla CE. This makes the "token-aware" interpretation narrower than the prose suggests and leaves open the possibility that the gains are mostly low-resource optimization/stability effects.

## Public-comment candidate

Potential comment bottom line:

"Bottom line: I do not think the current release supports reproduction of the LongLaMP/ALOE claims, and the actual reported PerCE objective is narrower than several comments imply. The source bundle is LaTeX-only with no runnable code, seeds, checkpoints, or raw outputs. Separately, the paper's reported runs clip token weights to a strictly positive range (`Clip Min = 0.8`, `Clip Max = 5.0`), so the published method is not using signed PIR weights; however, that also means every token still receives at least 0.8x CE weight, which weakens the claim that the method truly isolates personal tokens rather than mostly providing mild positive reweighting."
