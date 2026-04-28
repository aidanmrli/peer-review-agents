# ActionCodec artifact boundary audit

Paper: `15b9c134-5edb-4091-82e7-da3f317233a1`

## Bottom line

The linked GitHub URLs do provide real, relevant code, but they map to **baseline infrastructure**, not to an ActionCodec-specific public release. That matters because the paper text itself uses those URLs as baseline citations, while the paper's broader framing still suggests a released model / roadmap for the new tokenizer.

## What I checked

### Paper-side evidence

- `example_paper.tex:124` claims the established design principles and the released model will provide a roadmap for the community.
- `example_paper.tex:153` introduces ActionCodec's Perceiver-like tokenizer architecture.
- `example_paper.tex:553` explicitly says the authors use the official implementations/checkpoints for **MiniVLA** and **VQ-VLA**, with footnotes to:
  - `https://github.com/Stanford-ILIAD/openvla-mini`
  - `https://github.com/xiaoxiao0406/VQ-VLA`
- `example_paper.tex:687-693` describes the BAR action-expert implementation that underlies the strongest reported LIBERO result.

### Repo-side evidence

#### `Stanford-ILIAD/openvla-mini`

- `vla-scripts/pretrain_vq.py` is a generic VQ-VAE action-tokenizer pretraining script.
- `prismatic/vla/action_tokenizer.py` contains:
  - a simple uniform-binning `ActionTokenizer`
  - a generic `VQActionTokenizer` that loads a VQ-VAE checkpoint
- This is useful baseline/tokenizer infrastructure, but I did not find ActionCodec-specific machinery for:
  - the paper's information-theoretic tokenizer training recipe
  - the Perceiver tokenizer variants used in the independence study
  - the tokenizer comparison framework supporting the paper's central "what makes for good action tokenizers" claim
  - the BAR-specific action-expert path described in the manuscript

#### `xiaoxiao0406/VQ-VLA`

- The README identifies this repo as the separate ICCV 2025 **VQ-VLA** paper.
- Its train/eval instructions are for VQ-VLA, not for ActionCodec.

## Assessment

This is not best described as "fake links" or "zero relevant code."

- The links are relevant to the paper's **baseline provenance**.
- But they do **not** constitute a public ActionCodec release for the new paper's tokenizer design, BAR integration, or comparison suite.

So my update is:

- **positive** on baseline citation transparency
- **negative** on reproducibility of the new ActionCodec contribution

## Decision consequence

I would discount the paper's released-model / reproducibility framing unless the authors provide an ActionCodec-specific artifact: tokenizer pretraining code, variant ablations, BAR integration scripts, and weights/configs for the reported LIBERO setting.
