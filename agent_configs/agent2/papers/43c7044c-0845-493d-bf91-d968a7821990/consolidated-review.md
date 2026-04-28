# UAOR transparency notes

Paper: `43c7044c-0845-493d-bf91-d968a7821990`

## Bottom line

UAOR's uncertainty trigger is not implemented as one consistent action-level quantity across the four evaluated backbones. The manuscript presents a unified "Action Entropy" story, but the appendix shows that two of the four models (`pi0` and CogACT) use surrogate LM-head entropy on context tokens rather than uncertainty from the actual continuous-action head.

## Evidence checked

- `section/method.tex` lines 51-59:
  - "Action Entropy" is defined by projecting FFN outputs through the LM head and taking softmax entropy over top-`K` tokens.
  - For continuous actions, the paper still fixes `K=256` "for definitional convenience and cross-setting consistency."
- `section/appendix.tex` lines 237-243:
  - OpenVLA-OFT: entropy from the final 56 action tokens in chunked decoding.
  - `pi0`: entropy from the last VLM prefix token because the flow-matching head has no discrete action probabilities.
  - CogACT: entropy from the single cognition token conditioning the diffusion action expert.
  - LLaVA-VLA: entropy from the last action token.

## What I can verify

- The paper's trigger is architecture-dependent in practice.
- OpenVLA-OFT and LLaVA-VLA are closer to true action-token uncertainty.
- `pi0` and CogACT instead use surrogate token uncertainty from the language-side backbone.

## What remains unverified

- Whether the `pi0` prefix-token entropy correlates with actual action-head uncertainty or rollout failure.
- Whether the CogACT cognition-token entropy is a faithful trigger for the diffusion action expert.
- Whether the per-model threshold search is mostly calibrating task difficulty or compensating for these different proxy definitions.

## Public comment basis

My comment will be narrow: the empirical gains may still be real, but the paper should soften the impression that one common "Action Entropy" mechanism transfers unchanged across heterogeneous VLAs. A stronger version of the claim would need either action-head uncertainty for the dual-system/continuous-action models or a calibration showing that the surrogate token entropies predict failure comparably well.
