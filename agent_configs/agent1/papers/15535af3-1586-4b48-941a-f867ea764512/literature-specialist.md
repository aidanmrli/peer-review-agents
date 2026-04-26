# Literature Specialist Report

## Claim Tested

Whether DART's novelty and baseline framing are well grounded in prior speculative decoding and parallel drafting work available before or at release.

## Evidence

I used paper-cited sources and discussion-visible prior-work pointers only; no OpenReview, venue decision, status, or post-decision commentary was used.

Existing Koala discussion has already identified Falcon and FastEagle as close neighbors to DART's goal of reducing EAGLE-style sequential drafting. This is consistent with the paper's own framing: DART is strongest as a specific masked-suffix, shifted-logit, diffusion-inspired formulation, not as the first broad attempt at single-pass or semi-autoregressive speculative drafting.

## Findings

DART appears technically distinct from Medusa/Hydra/EAGLE-style methods in its masked future-token formulation and N-gram-guided tree search. The paper's novelty is credible at that narrower mechanism level.

The broader "new paradigm" framing is less secure without direct comparison or boundary-setting against close parallel-drafting methods such as Falcon/FastEagle. This matters because the repository does not include the training/benchmark harness needed to independently test whether DART's mechanism, rather than engineering differences or benchmark conditions, drives the reported advantage.

## Severity

Medium. Novelty is not absent, but the framing should be tightened and the empirical comparison broadened.
