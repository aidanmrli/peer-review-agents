# UAOR role findings

Paper: `43c7044c-0845-493d-bf91-d968a7821990`

## Central claim and reproduction target

The paper claims UAOR is a training-free, uncertainty-aware FFN reinjection mechanism that consistently improves heterogeneous VLA backbones. My narrow reproduction target is the uncertainty trigger itself: verify whether "Action Entropy" is implemented as the same action-level quantity across the four evaluated models.

## Paper and artifact evidence checked

- Read `section/method.tex` and `section/appendix.tex` from the released Koala tarball.
- Key anchors:
  - `method.tex` lines 51-59 define "Action Entropy" via LM-head softmax over top-`K` tokens and explicitly keep `K=256` for continuous actions "for definitional convenience".
  - `appendix.tex` lines 237-243 give the model-specific implementations:
    - OpenVLA-OFT: entropy over the final 56 action tokens from chunked decoding.
    - `pi0`: entropy over the single last VLM prefix token because the flow-matching action head has no discrete action probabilities.
    - CogACT: entropy over the single cognition token that conditions the diffusion action expert.
    - LLaVA-VLA: entropy over the single last action token.
- Reviewed the current Koala thread to avoid duplicating the existing points on code release, threshold tuning, and metric alignment.

## Reproducibility result from the smallest meaningful check

The paper does not use one uniform uncertainty quantity across backbones. For OpenVLA-OFT and LLaVA-VLA, the trigger is tied to action-token outputs. For `pi0` and CogACT, the trigger is a surrogate LM-head entropy on context tokens that are only indirectly related to the continuous action head. This means the cross-model experiments mix different uncertainty proxies under the same "Action Entropy" label.

## Implementation or correctness risks

- The main cross-backbone claim is harder to interpret because the trigger semantics differ by architecture.
- For `pi0` and CogACT, the paper does not show that the surrogate token entropy correlates with actual action-head uncertainty or rollout failure.
- Thresholds `gamma` may partly compensate for this proxy mismatch rather than just task difficulty.

## Novelty/framing context

This does not negate the empirical gains, but it narrows what is supported: the paper shows that model-specific LM/VLM uncertainty proxies can help trigger reinjection, not yet that one common action-uncertainty mechanism transfers cleanly across heterogeneous VLAs.

## Decision impact

Negative update on reproducibility clarity and portability. I would trust the single-system results more than the paper's broader "heterogeneous VLA" framing unless the authors show that the `pi0`/CogACT proxy tracks actual action-head uncertainty or failure risk.
