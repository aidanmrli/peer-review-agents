# CER Artifact Audit

Paper: `6454dcf3-6eff-4b23-b5be-9bfaa905a83a`

## Scope

This note documents a narrow reproducibility audit of the released CER artifact against the paper source.

## Evidence checked

- Paper source: `tmp/6454dcf3_src/example_paper.tex`
- Repo: `https://github.com/changyi7231/CER`
- Cloned commit observed locally on 2026-04-29: `ec556969a33a8123950e81f93ca63bdec039bbd5`

## Findings

1. The release is real code, not an empty placeholder.
   - `recipe/cer/src/reward_manager.py` implements CER-related reward plumbing.
   - `recipe/cer/src/data_preparation.py` supports the named datasets from the README.
   - `recipe/cer/run.sh` launches PPO training through the released stack.

2. The released experiment recipe does not match the full paper matrix.
   - The paper reports results for both `Qwen3-4B-Base` and `Qwen3-8B-Base` in Tables 1 and 2.
   - The released launcher is hard-coded to `model_path="Qwen/Qwen3-8B-Base"`.
   - I did not find a released parallel launcher/config for the 4B setting.

3. The released training recipe is WebInstruct-specific.
   - The paper says models are trained on both WebInstruct and MATH-7.5K.
   - The visible launcher trains from `TIGER-Lab/WebInstruct-verified/train_repeated.parquet`.
   - I did not find a released MATH-7.5K training launcher matching the second reported regime.

4. The hardware/runtime setup does not line up with Table 3.
   - The paper states Table 3 runtimes were measured on four NVIDIA H100 GPUs.
   - The released launcher sets `n_gpus_per_node=8`.
   - This makes the runtime table hard to independently audit from the public recipe.

5. The comparison matrix is not packaged as runnable manifests.
   - The paper compares CER against Exact-match, Rule, VeriFree, General-verifier, and Rule+CER.
   - I did not find released paper-matched launchers/configs for those variants or for the table-by-table runs.

## Bottom line

The artifact is substantially better than a placeholder release, but it is not yet packaged tightly enough to reproduce Tables 1-3 as written. My confidence reduction is therefore about experiment reproducibility and runtime auditability, not about whether CER itself has been implemented at all.
