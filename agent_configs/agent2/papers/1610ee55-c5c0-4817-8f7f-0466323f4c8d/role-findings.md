## Reproducibility lead

Central claim and reproduction target: knowledge-graph path-derived rewards, added after grounded SFT, improve compositional medical reasoning and let Qwen3 14B outperform larger frontier models on harder ICD-Bench questions. My reproduction target this cycle was narrower: verify whether the released artifact is sufficient to reproduce the SFT to RL pipeline and the paper's key ablation logic.

## Reproducer A

Artifact-first check: cloned `https://github.com/jha-lab/kg-implicit-reward-compositional-rl` and inspected `README.md`, `sft_training.py`, `rl_training.py`, `data_prep.py`, `data_loader.py`, and `create_filtered_dataset.py`. The repo contains training code and a DeepSpeed config, but not the actual training data, KG, filtered 5k RL split, checkpoints, logs, or run manifests. `README.md` explicitly states the actual training data is not included and that the release assumes a QA-GNN-style schema with fields such as `question_and_explanation`, `paths`, `category`, `source_concept`, and `target_concept`. `data_loader.py` and `sft_training.py` still use placeholder dataset paths like `/path/to/your/...`.

## Reproducer B

Clean-room/specification check: extracted the paper source from the tarball and read the ablation and methods sections in `source/arxiv_main_icml.tex`. The paper does isolate SFT versus RL on the 8B model: SFT baseline 70.86%, path alignment only 79.29%, binary only 79.46%, and path-alignment plus negative binary 82.20%. The intended split is 19.66k SFT examples plus a 5k RL subset, and the paper states the final 14B runs used 8x H200 GPUs. So the algorithmic story is more explicit than the current thread suggests, but the release is still missing the dataset and run artifacts needed to verify that story.

## Implementation auditor

Code and paper are directionally aligned on the two active final rewards. In `rl_training.py`, the active rewards are `correctness_reward_func` and `path_alignment_reward_func`; the code implements asymmetric correctness scoring (+0.1 for correct, -1.0 for wrong/missing) and a token-overlap path reward capped at 1.5. Optional "thinking quality" and semantic similarity rewards remain in code but are not active by default, matching the paper's appendix discussion that "all rewards" hurts performance. The main artifact mismatch is completeness, not a direct code-paper contradiction.

## Correctness specialist

The public evidence supports the claim that the authors thought about SFT versus RL ablations. What remains unverified is the headline empirical generalization claim: no public processed ICD-Bench split, no KG/path construction release, no checkpoints, and no training/eval logs showing the reported 14B frontier-model comparisons or option-shuffling stress tests. Because the path reward depends on the exact serialized KG paths, the missing data artifacts are load-bearing rather than incidental.

## Literature specialist

The framing around KGs as implicit reward models is interesting and somewhat distinct from plain answer-level RL or pure CoT distillation. My contribution this cycle is not a novelty judgment against external post-publication signals; it is that the release currently substantiates "we have a plausible training recipe" more than "the reported 14B benchmark gains are independently reproducible."
