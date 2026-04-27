# Review Notes: 1610ee55-c5c0-4817-8f7f-0466323f4c8d

Paper: Knowledge Graphs are Implicit Reward Models: Path-Derived Signals Enable Compositional Reasoning

Date: 2026-04-26

Bottom line: the paper does provide a meaningful SFT-versus-RL ablation in the appendix, but the public artifact is still too incomplete to independently verify the headline 14B compositional-reasoning gains.

Evidence gathered:

1. Paper source audit
   - Extracted the submitted source tarball and inspected `source/arxiv_main_icml.tex`.
   - Appendix B reports an explicit SFT baseline and SFT+RL ablations on Qwen3 8B: SFT baseline 70.86%, path alignment only 79.29%, binary only 79.46%, and path-alignment plus negative binary 82.20%.
   - The paper states the main recipe uses 19.66k SFT examples plus a 5k RL subset and that the final 14B runs use 8x H200 GPUs.

2. Artifact-first audit
   - Cloned `https://github.com/jha-lab/kg-implicit-reward-compositional-rl`.
   - The repo contains generic training scripts (`sft_training.py`, `rl_training.py`, `data_prep.py`, `create_filtered_dataset.py`) and DeepSpeed/SLURM scaffolding.
   - `README.md` explicitly says the actual training data is not included and that the release assumes a QA-GNN-style KG-derived schema.
   - `data_loader.py`, `sft_training.py`, and `rl_training.py` still contain placeholder dataset/checkpoint paths such as `/path/to/your/tokenized_dataset`, `/path/to/your/rl_dataset`, and `./sft_models/model-lora/checkpoint-XXX`.
   - No processed ICD-Bench/medical KG data, no released filtered 5k RL split, no model checkpoints, and no training/evaluation logs were present.

3. Code-paper consistency check
   - The final active rewards in `rl_training.py` are `correctness_reward_func` and `path_alignment_reward_func`, which is consistent with the paper's claimed final setup.
   - Optional thinking-quality and semantic-similarity rewards exist in code but are disabled by default, also matching the appendix discussion that combining all rewards degrades performance.

Independent-pass summary:

- Pass A (artifact-first): I could verify that the released code sketches the intended SFT to GRPO pipeline, but I could not recreate the reported setup because the KG-dependent data artifacts and exact run products are missing.
- Pass B (paper-first): I could recover the intended ablation logic and confirm that the paper already addresses the "is RL helping beyond SFT?" question on the 8B model.

Decision impact:

- Positive: the ablation evidence makes the causal story more credible than a purely descriptive abstract would suggest.
- Negative: the current release does not support an independent rerun of the central 14B empirical claim, especially because path reward computation depends on the unreleased serialized KG paths and filtered train/RL split.

What would change my assessment:

- Release the processed 19.66k/5k data split or a reproducible converter from the underlying KG benchmark to the exact training format.
- Release at least one runnable eval manifest or log bundle for the reported 14B experiments, including the option-shuffling stress test and frontier-model comparison protocol.
