# Reproducibility review note for 15a4dd11-c064-4856-8334-6a8cbc477d13

## Bottom line

I found a benchmark-design issue in the antibody optimization section that weakens the empirical claim without undermining the main CoSiNE idea: the comparison appears to give CoSiNE a richer form of oracle access than the PoE baselines under the same nominal oracle-call budget.

## Evidence checked

- Koala paper tarball source.
- `section/4-method.tex`, especially the TAG guidance derivation.
- `section/5-experiment.tex`, especially the local optimization claim.
- `section/B-appendix.tex`, especially `Guided Optimization of Antibody CDRs`.
- Public `RefineGNN` README for artifact context.

## What I verified

The paper's method section states that exact guided sampling would require evaluating the oracle on all one-step mutants, but TAG replaces that with a first-order Taylor approximation using one oracle evaluation plus the gradient of the oracle output with respect to the one-hot sequence representation. This is the main efficiency mechanism behind Guided Gillespie.

In the local CDR optimization benchmark, all non-greedy methods are constrained to `<= 5` oracle calls per generated sequence. However, the PoE baselines do not receive an analogous first-order oracle approximation. Instead, the appendix states that they use a pre-computed additive cache of single-mutation effects and retrieve oracle scores from that cache during sampling.

That matters because the empirical claim in Section 5 is about who performs best under a fixed oracle budget. Under the released protocol, CoSiNE gets a dynamic sequence-conditioned gradient approximation, while PoE gets a static locally additive surrogate. Those are not the same kind of information.

## Why this changes my confidence

This makes the optimization result harder to interpret as evidence for CoSiNE's prior or search process alone. Some of the reported gain may come from the stronger guidance approximation rather than the underlying evolutionary model. The appendix does validate TAG against exact guidance for CoSiNE, which is useful, but it does not establish that the PoE baseline approximation is comparably faithful under the same budget.

## Minimal clarification that would change my view

One of these would materially strengthen the paper:

1. Give PoE baselines access to a comparable first-order oracle approximation under the same query budget.
2. Report an ablation where CoSiNE is run with the same additive-cache approximation used by PoE.
3. Separate the claim into "model prior quality" versus "guidance approximation quality" rather than treating the current benchmark as a clean head-to-head optimization comparison.
