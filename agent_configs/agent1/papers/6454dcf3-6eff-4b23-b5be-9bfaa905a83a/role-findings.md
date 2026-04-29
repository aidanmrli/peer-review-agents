# CER Reproducibility Findings

## Central claim and reproduction target

The paper claims CER is a practical reward mechanism that improves RL training across both WebInstruct and MATH-7.5K, for both Qwen3-4B-Base and Qwen3-8B-Base, with runtime comparisons in Table 3 reported on four NVIDIA H100 GPUs.

## Paper and artifact evidence checked

- Paper source: `tmp/6454dcf3_src/example_paper.tex`
- Repo clone: `tmp/cer_repo`
- Key files inspected:
  - `tmp/cer_repo/README.md`
  - `tmp/cer_repo/recipe/cer/run.sh`
  - `tmp/cer_repo/recipe/cer/src/reward_manager.py`
  - `tmp/cer_repo/recipe/cer/src/data_preparation.py`

## Reproducibility result from the smallest meaningful check

I verified that the release does include core CER code rather than a placeholder: the repo contains CER-specific reward code and data-preparation scripts, and `run.sh` launches PPO training through the released pipeline.

However, the released recipe is not paper-complete:

- `recipe/cer/run.sh` is hard-coded to `Qwen/Qwen3-8B-Base`, not the paper's full 4B+8B matrix.
- The only released training launcher uses `TIGER-Lab/WebInstruct-verified/train_repeated.parquet`; I did not find a released MATH-7.5K training launcher matching the paper's second training regime.
- The released launcher sets `n_gpus_per_node=8`, while the paper says Table 3 runtimes were measured on four H100 GPUs.
- I did not find released experiment manifests for the reported Exact-match / Rule / VeriFree / General-verifier / Rule+CER comparisons in Tables 1-3.

## Implementation or correctness risks

- The artifact supports "CER exists in code" but not "the reported tables can be rerun from released configs."
- Hardware and training-regime mismatches make the runtime table especially hard to audit.
- The release currently exposes one main CER recipe rather than a paper-matched suite of experiment manifests.

## Novelty/framing context

This is not a claim that the method is unreleased. It is a narrower reproducibility calibration: the code is materially present, but the experiment packaging is incomplete relative to the paper's reported comparison matrix.

## Decision impact

I would treat this as a confidence reduction on empirical reproducibility, not a refutation of CER's underlying idea. A stronger release would include paper-matched launchers/configs for:

- WebInstruct and MATH-7.5K
- Qwen3-4B and Qwen3-8B
- the verifier baselines and Rule+CER combination
- the four-H100 runtime setup used for Table 3
