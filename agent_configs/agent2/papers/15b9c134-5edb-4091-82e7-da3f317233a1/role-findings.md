## Central claim and reproduction target

The paper claims to identify ActionCodec-specific tokenizer design principles for VLA optimization and to provide a released model that supports the reported LIBERO gains, including the BAR variant and tokenizer comparisons.

## Paper and artifact evidence checked

- Paper source: `example_paper.tex`.
- Key paper locations:
  - Abstract claim that the released model will provide a roadmap and 97.4% SOTA without robotics pre-training (`example_paper.tex:124`).
  - ActionCodec architecture and Perceiver framing (`example_paper.tex:153`).
  - Baseline-artifact footnotes for MiniVLA and VQ-VLA (`example_paper.tex:553`).
  - BAR implementation description (`example_paper.tex:687-693`).
- Linked repo 1: `Stanford-ILIAD/openvla-mini`.
  - Generic VQ pretraining script in `vla-scripts/pretrain_vq.py`.
  - Generic `ActionTokenizer` / `VQActionTokenizer` in `prismatic/vla/action_tokenizer.py`.
- Linked repo 2: `xiaoxiao0406/VQ-VLA`.
  - README identifies it as the ICCV 2025 VQ-VLA paper and provides VQ-VLA-specific training/eval instructions.

## Reproducibility result from the smallest meaningful check

The linked artifacts partially support baseline provenance but do not expose an ActionCodec-specific release path.

- Positive: the paper text itself presents `openvla-mini` and `VQ-VLA` as baseline implementations, and the repos do contain generic/action-tokenizer infrastructure consistent with that role.
- Negative: I did not find an ActionCodec-specific tokenizer implementation covering the paper's distinctive elements: the information-theoretic training recipe, the Perceiver-based tokenizer variants used for the independence ablation, the ActionCodec tokenizer comparison framework, or the BAR-specific action-expert evaluation path claimed in the paper.

## Implementation or correctness risks

- The Koala metadata can be misread as paper-artifact links, but the paper source suggests they are baseline links. That softens the "misleading link" criticism.
- The stronger reproducibility issue remains: the paper claims a released model / roadmap, yet the public URLs I checked only substantiate prior or generic tokenizer infrastructure, not the new ActionCodec contribution.
- This makes it hard to verify whether the reported improvements come from the claimed tokenizer principles, the BAR stack, or unreleased integration details.

## Novelty/framing context

The source positions the linked repos as borrowed baselines, not as the ActionCodec release itself. So the artifact story is more nuanced than "wrong repo": the citations are understandable, but the new paper-specific artifact is still missing.

## Decision impact

Positive update on baseline citation transparency; negative update on ActionCodec reproducibility completeness. This does not invalidate the idea, but it lowers confidence in the paper's released-model and independent-verification story.
