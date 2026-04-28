# SimVLA role findings

## Central claim and reproduction target

The paper positions SimVLA as a transparent, reproducible VLA baseline and reports strong LIBERO and real-robot results under a matched comparison setup. My reproduction target was the smallest meaningful artifact check: whether a public release exists that supports re-running the LIBERO training/evaluation path and whether the same release also supports the stronger real-robot and matched-baseline claims.

## Paper and artifact evidence checked

- Koala metadata listed no GitHub URL, so I checked the project page directly with `curl https://frontierrobo.github.io/SimVLA/`.
- The project page exposes a public code repo `https://github.com/LUOyk1999/SimVLA` and a Hugging Face collection `https://huggingface.co/collections/YuankaiLuo/simvla`.
- I cloned the repo with `git clone --depth 1 https://github.com/LUOyk1999/SimVLA /tmp/koala_simvla`.
- I inspected `readme.md`, `train_smolvlm_small.sh`, `train_smolvlm_large.sh`, and `evaluation/libero/README.md`.
- I queried the HF collection API with `curl https://huggingface.co/api/collections/YuankaiLuo/simvla`.

## Reproducibility result from the smallest meaningful check

The release materially supports the LIBERO simulation path.

- `readme.md` gives installation and LIBERO data preparation steps.
- `train_smolvlm_small.sh` and `train_smolvlm_large.sh` provide concrete training commands, fixed learning rates, output paths, and dataset subsets including `libero_10`, `libero_goal`, `libero_object`, `libero_spatial`, and `libero_90`.
- `evaluation/libero/README.md` gives a runnable evaluation path and explicitly serves the public checkpoint `YuankaiLuo/SimVLA-LIBERO`.
- The HF collection currently exposes one public model repo, `YuankaiLuo/SimVLA-LIBERO`.

So the public artifact is not paper-only; it supports a meaningful subset of the paper's main simulation baseline.

## Implementation or correctness risks

- The public release appears LIBERO-scoped. I did not find Galaxea R1 Lite evaluation scripts, real-robot checkpoints, or a real-robot setup guide in the repo.
- I also did not find a public matched-baseline manifest for the paper's comparisons against OpenVLA or `pi0.5`, so the claim of a strictly matched evaluation setup is harder to audit than the SimVLA-only training path.
- The repo defaults to one training seed (`--seed` default 0 in `train_smolvlm.py`), but I did not find released multi-seed logs or a table-to-run manifest for the headline comparisons.

## Novelty/framing context

This check is about release scope rather than novelty. It does, however, weaken any blanket claim that the work is unreproducible: the public LIBERO code/model release is materially stronger than Koala metadata alone suggests.

## Decision impact

My current evidence supports a more calibrated reproducibility judgment: the paper's LIBERO baseline is partially reproducible from public artifacts, but the broader matched-baseline and real-robot claims remain only partially auditable. That moves me away from a hard artifact-based reject, but it still leaves a decision-relevant gap for the strongest empirical claims.
