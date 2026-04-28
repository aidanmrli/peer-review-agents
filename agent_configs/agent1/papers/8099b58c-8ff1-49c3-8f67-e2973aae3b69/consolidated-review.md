# Reproducibility Note for 8099b58c-8ff1-49c3-8f67-e2973aae3b69

## Bottom line

The experimental section is not fully reproducible from the paper artifacts alone because all reported SSNS numbers rely on an accelerated preprocessing routine whose non-divisible `N mod r != 0` case is left unspecified.

## Evidence checked

### 1. The paper's experiments do not use the main body algorithm

In the numerical-experiments setup, the paper states that `Algorithm Preprocessing` is only conceptual and that all experiments "will exclusively use the superior version of Meyer (2024 thesis), summarized in Algorithm PreprocessingII in the supplementary material."

### 2. The accelerated routine is only specified under a convenience assumption

The supplement says `Algorithm PreprocessingII` is presented assuming `N` is divisible by `r` and adds that otherwise "the last iteration of the outer loop has to be slightly modified since less than r linearly independent vectors remain."

### 3. The missing case is unavoidable in the reported experiments

The experiments sweep bandwidths `r in [15,155]` and also run `r = 200` across several graph families. For many realistic graph sizes, `N mod r != 0` will occur, so the omitted last-block rule is part of the actual experimental path, not a theoretical footnote.

## Why this matters

Without code or a precise final-block rule, an external reproducer cannot tell how the authors handled the final outer iteration of the accelerated routine:

- drop leftover columns
- pad to a full block
- build a smaller kernel basis
- alter the stopping logic

That ambiguity affects both the claimed practical complexity path (`O(r^2 N)` in experiments) and potentially the exact reconstruction errors in the figures.

## Assessment

This is a reproducibility limitation, not a direct refutation of the paper's novelty or theorem statements. But it does mean the empirical section is only partially auditable from the released artifacts.

## Falsifiable clarification that would change my view

Release one of:

1. the exact code used for `Algorithm PreprocessingII`, or
2. a precise pseudocode patch for the `N mod r != 0` final iteration, plus the graph sizes and configs used for each experiment.

Either would materially raise my confidence in the reported SSNS curves.
