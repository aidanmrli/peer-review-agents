# Expert Threshold Routing - Independent Reproducer B

- Paper ID: `acca775c-254b-410c-9252-c37ed998431f`
- Title: `Expert Threshold Routing for Autoregressive Language Modeling with Dynamic Computation Allocation and Load Balancing`
- Assigned role: Independent Reproducer B
- Date: 2026-04-25

## Task scope

Independently reconstruct the core claim through static code and configuration tracing rather than attempting the same execution route as Reproducer A.

## Claim attempted

The paper's ET configuration uses 16 routed experts with `G=1,E=16`, plus one shared expert, so that each token activates the shared expert and on average one routed expert (`artifacts/v2.tex:304-309`, `artifacts/v2.tex:715-761`). The released code/configs should encode this experimental setup and the EMA-threshold routing rule.

## Independent route used

I traced the released config composition and model construction:

- `configs/mlp/et.yaml`
- `configs/mlp/ec.yaml`
- `configs/model_size/d20.yaml`
- `configs/training/standard.yaml`
- `src/models/model_base.py`
- `src/models/expert_threshold_choice.py`
- `src/models/engines/common.py`
- `train.py`

## Trace findings

The mechanism is partially present:

- `src/models/expert_threshold_choice.py:29-30` defines a unified EC/ET routed-expert MLP.
- `src/models/expert_threshold_choice.py:97-99` uses top-k during training until the selection policy is switched, and threshold routing during evaluation.
- `train.py:181-191` switches to threshold routing at the configured warmup step.
- `src/models/engines/common.py:43-53` accumulates top-k cutoffs.
- `src/models/engines/common.py:66-120` applies threshold routing and capacity clamping.
- `src/models/engines/common.py:251-272` computes bias-corrected cutoff EMA updates.

The exact paper configuration is not present:

- The paper states `G=1,E=16` plus a shared expert (`artifacts/v2.tex:307`, `artifacts/v2.tex:717`, `artifacts/v2.tex:761`).
- The released ET config uses `granularity: 2` and `expansion: 8` (`configs/mlp/et.yaml:4-19`).
- The released EC config also uses `granularity: 2` and `expansion: 8` (`configs/mlp/ec.yaml:4-18`).
- The model asserts that shared expert variants require `granularity >= 2` (`src/models/model_base.py:123-130`).
- With shared experts, the code computes the target per-expert token count as `n_tokens * (g - 1) // (g * e)` (`src/models/engines/common.py:13-21`). Thus the released `G=2,E=8` gives one routed expert per token on average, but the paper-stated `G=1,E=16` would make `g-1=0` and is also rejected by the assertion.

## Observed result

Partial match for method logic, mismatch for exact experiment reconstruction. The code likely intends `G=2,E=8` to produce 16 routed experts plus one shared expert with half-width experts, matching some qualitative aspects of the text, but the notation/configuration in the paper is not the runnable released setting.

## Agreement with Reproducer A

This agrees with Reproducer A's conclusion that the headline empirical claim is not independently reproduced. My route adds a separate blocker: even before running, the paper's stated MoE configuration cannot be exactly instantiated from the released config/code.

## Limitations

I did not execute a synthetic forward pass because the local environment lacks PyTorch. The static trace is still sufficient to establish the paper-code mismatch.

## Confidence level

High for the configuration mismatch and partial method-code match.

## Decision impact

Major. Exact configuration ambiguity is decision-relevant because the reported parameter counts, active compute, load-balancing target, and per-expert capacity all depend on `G`, `E`, and whether the shared expert is included in the target-rate formula.
