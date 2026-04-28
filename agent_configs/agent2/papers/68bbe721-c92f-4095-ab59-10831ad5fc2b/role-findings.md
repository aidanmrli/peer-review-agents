# AgentScore role findings

## Central claim and reproduction target

AgentScore claims that an LLM-guided but deterministically validated pipeline can build deployable unit-weighted clinical checklists that outperform prior score-learning baselines on eight tabular healthcare tasks and two external-validation settings.

## Paper and artifact evidence checked

- Read current Koala thread for paper `68bbe721-c92f-4095-ab59-10831ad5fc2b`.
- Downloaded and unpacked the Koala tarball source.
- Checked `sections/appendix.tex` around the train/validation/test split, statistical-analysis, PLR baseline, and algorithmic-detail sections.
- Key paper locations inspected:
  - `sections/appendix.tex:713-722` for cross-validation and AUROC handling.
  - `sections/appendix.tex:787-805` for PLR baseline construction.
  - `sections/appendix.tex:1018-1032` for the statistical comparison protocol over `n=40` pooled fold-level observations.
  - `sections/appendix.tex:1344-1348` for code-release status.

## Reproducibility result from the smallest meaningful check

I verified a concrete evaluation-policy detail that is material to the headline comparison:

- The appendix states that when a baseline is missing or degenerates on a fold, its AUROC is set to `0.5` to preserve pairing.
- The same appendix reports paired significance tests on pooled fold-level AUROCs across eight datasets and five folds (`n=40`), with Holm-Bonferroni correction.
- The paper does not disclose how many folds required this `0.5` fallback for each baseline.

This is enough to conclude that part of the reported statistical advantage could come from fallback handling rather than only from superior checklist construction.

## Implementation or correctness risks

- If degenerate baseline folds are frequent, imputing `0.5` can materially widen AgentScore-vs-baseline gaps.
- Pooling folds across heterogeneous datasets into a single `n=40` paired test can obscure whether wins are broad or concentrated in datasets where baselines collapse.
- Full code is not public yet; the appendix says it will be released only upon acceptance, so I cannot audit how often the fallback path is triggered in practice.

## Novelty or framing context

This finding does not negate the paper's framing contribution around deployable unit-weighted checklists. It does narrow confidence in the empirical strength of the superiority claim until the authors disclose baseline-collapse frequency and rerun the comparison under a sensitivity analysis that excludes or separately tabulates fallback folds.

## Decision impact

My confidence in the empirical margin over baseline score learners should be discounted unless the authors report:

1. The number of folds/datasets where each baseline was assigned AUROC `0.5`.
2. A sensitivity analysis excluding those folds or reporting per-dataset wins without pooled fallback-heavy comparisons.

Absent that clarification, the paper still has an interesting problem formulation, but the strength of the experimental evidence is weaker than the headline claim suggests.
