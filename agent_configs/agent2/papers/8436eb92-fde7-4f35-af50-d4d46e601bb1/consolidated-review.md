# SimVLA consolidated review

## Bottom line

The public artifact supports the LIBERO simulation baseline better than the Koala metadata suggests, but it does not yet fully support the paper's stronger matched-baseline and real-robot reproducibility claims.

## Evidence checked

- Project page fetched from `https://frontierrobo.github.io/SimVLA/`.
- Public code repo cloned from `https://github.com/LUOyk1999/SimVLA`.
- Hugging Face collection queried at `https://huggingface.co/api/collections/YuankaiLuo/simvla`.
- Files inspected:
  - `readme.md`
  - `train_smolvlm_small.sh`
  - `train_smolvlm_large.sh`
  - `evaluation/libero/README.md`
  - `train_smolvlm.py`

## What the release does support

- The project page links a public GitHub repo and a public Hugging Face collection.
- The repo includes concrete LIBERO data-prep, training, and evaluation paths.
- The evaluation README serves a public checkpoint, `YuankaiLuo/SimVLA-LIBERO`.
- The training scripts expose specific hyperparameters and dataset subset choices rather than leaving the main simulation recipe implicit.

## What remains hard to verify

- I did not find a public Galaxea R1 Lite evaluation path, real-robot checkpoint, or real-robot setup instructions.
- I did not find a public manifest that ties the matched OpenVLA / `pi0.5` comparison tables to exact checkpoints, seeds, and hyperparameter settings.
- I did not recover multi-seed result logs or a table-to-run mapping for the headline comparisons.

## Decision consequence

This release is materially stronger than a paper-only submission and does support a meaningful LIBERO reproducibility check. However, the strongest claims in the paper still rely on evidence that is not yet fully auditable from the public artifact. My assessment is therefore: supports the core LIBERO baseline only partially, and cannot yet verify the broader matched-baseline and real-robot story.
