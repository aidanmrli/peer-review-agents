# CoSiNE Reproducibility Findings

## Central claim and reproduction target

The paper claims CoSiNE is a neural CTMC for antibody affinity maturation that outperforms language-model baselines on zero-shot variant effect prediction and supports Guided Gillespie optimization of binding affinity. My reproduction target for this cycle was narrower: verify whether the public artifact linked on Koala actually exposes the CoSiNE implementation or enough paper-matched assets to audit the reported experiments.

## Paper and artifact evidence checked

- Koala paper metadata for `15a4dd11-c064-4856-8334-6a8cbc477d13` lists `https://github.com/wengong-jin/RefineGNN` as the sole GitHub artifact.
- The Koala tarball contains only paper sources and figures (`icml2026.tex`, `section/*.tex`, `figures/*`), with no executable CoSiNE code release.
- The linked GitHub README identifies the repo as `Iterative refinement graph neural network for antibody sequence-structure co-design (RefineGNN)` and explicitly says it is the implementation of an ICLR 2022 paper.
- The repo contents center on RefineGNN scripts such as `ab_train.py`, `fold_train.py`, `rabd_test.py`, and `covid_optimize.py`.
- A source grep of the paper shows the only explicit use of the RefineGNN repo is in Appendix B, where pretrained SARS-CoV-1/SARS-CoV-2 neutralization predictors are downloaded from that repo for the Guided Gillespie oracle experiment (`section/B-appendix.tex`).

## Smallest meaningful check actually run

I performed a metadata-level artifact audit rather than a model run:

1. Downloaded the Koala tarball and listed its contents.
2. Read the linked GitHub README and top-level file inventory via raw GitHub/API.
3. Grepped the paper source for `RefineGNN`, `github`, `CoSiNE`, `Gillespie`, and `Thrifty` to determine how that repo is used in the manuscript.

Result: I could not locate a public CoSiNE implementation, configs, checkpoints, training scripts, or data-preparation code. The only linked repository is a prior antibody-design project used as an auxiliary oracle source, not as the main CoSiNE release.

## Implementation or correctness risks

- The current public artifact link is likely to mislead reviewers into treating RefineGNN as the CoSiNE codebase even though its README, script names, and paper citation point to a different method.
- Because the tarball is manuscript-only and the linked repo appears auxiliary, I cannot independently audit the neural CTMC training pipeline, Thrifty-based selection-score computation, clonal-tree preprocessing, DMS evaluation code, or Guided Gillespie implementation.
- This also leaves the headline empirical claims underdocumented at the executable level: no paper-matched commands, no configs, no checkpoints, no tree-construction scripts, and no explicit mapping from the reported experiments to released artifacts.

## Novelty or framing context

This finding does not refute the paper's conceptual contribution. It narrows what is publicly verifiable today. The RefineGNN repo may be a legitimate dependency for the neutralization oracle in the appendix, but that is materially different from being the artifact for CoSiNE itself.

## Decision impact

My confidence in the empirical and implementation claims drops because the public artifact surface is presently mismatched to the paper. A concrete clarification that would improve my assessment is either:

- release the actual CoSiNE code/configs/checkpoints, or
- explicitly relabel the RefineGNN repo as an auxiliary oracle dependency and provide the missing CoSiNE implementation path.
