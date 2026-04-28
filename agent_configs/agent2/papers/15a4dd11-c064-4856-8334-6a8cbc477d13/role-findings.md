# Central claim and reproduction target

The paper claims CoSiNE improves zero-shot antibody variant effect prediction and enables guided antibody optimization under tight oracle budgets. My smallest meaningful check was to audit whether the optimization protocol supports a fair, reproducible comparison between CoSiNE and the reported guided baselines.

# Paper and artifact evidence checked

- Submission PDF source tarball from Koala.
- `section/4-method.tex` for the exact TAG guidance formulation.
- `section/5-experiment.tex` for the optimization claims.
- `section/B-appendix.tex` for the optimization protocol details.
- Public README of `https://github.com/wengong-jin/RefineGNN` to confirm what artifact is actually exposed.

# Reproducibility result from the smallest meaningful check actually run

I could reproduce the paper's stated optimization protocol at the documentation level, and that protocol exposes a decision-relevant comparability gap:

- CoSiNE's TAG guidance is explicitly derived to replace `L x (|A|-1) + 1` oracle queries with one oracle evaluation plus a gradient-based first-order approximation around the current sequence (`section/4-method.tex`, Eq. 9 discussion).
- In the constrained local optimization experiment, all non-greedy methods are capped at `<= 5` oracle calls per generated sequence (`section/B-appendix.tex`).
- Under that cap, the PoE baselines do not use the same first-order oracle access. Instead, they pre-compute single-mutation effects once and reuse an additive cache during Gibbs sampling, which the paper states "approximates the fitness landscape as locally additive" (`section/B-appendix.tex`).

So the reported comparison is not just "same oracle budget, different sequence prior." It is "CoSiNE gets a sequence-conditioned gradient approximation of the oracle during search, while PoE baselines are limited to a static additive cache."

# Implementation or correctness risks

- This makes the local optimization result hard to attribute cleanly to CoSiNE's evolutionary prior versus a stronger guidance mechanism.
- The paper reports that Guided Gillespie beats PoE on predicted affinity while preserving humanness, but the baseline handicap is algorithmic, not only computational.
- The auxiliary appendix comparison of exact guidance versus TAG (`section/C-appendix.tex`) validates TAG against exact guidance for CoSiNE itself; it does not show that the PoE approximation is equally faithful under the same budget.

# Novelty or framing context

The core CTMC-plus-neural parameterization is still novel enough to matter. My concern is narrower: the optimization benchmark seems to compare unlike guidance access patterns, so it is weaker evidence for the paper's practical design claim than the VEP section.

# Decision impact

This does not refute the main modeling idea. It does lower my confidence in the optimization evidence, especially the claim that CoSiNE is the strongest budget-constrained guided generator. A cleaner comparison would either give baselines access to equivalent first-order oracle information or report an ablation separating model prior from guidance approximation quality.
