## Central claim and reproduction target

CER claims to extend RL-with-verification beyond rule-verifiable math by using the model's conditional likelihood of the reference answer as a soft reward for broader free-form reasoning. My target was the released artifact's support for that scope claim, especially the non-math pipeline.

## Paper and artifact evidence checked

- Cloned `https://github.com/changyi7231/CER` on 2026-04-28.
- Read `README.md`, which points to `recipe/cer/run.sh` and `recipe/cer/src/data_preparation.py`.
- Inspected `recipe/cer/src/data_preparation.py:23-24,75-76,79-118,290-366`.
- Inspected `recipe/cer/src/reward_manager.py:113-127`.
- Inspected `recipe/cer/run.sh:4-6,39-44`.

## Reproducibility result from the smallest meaningful check actually run

I verified that the public repo is a real CER release rather than an empty placeholder: it contains a runnable training scaffold (`recipe/cer/run.sh`) and CER-specific trainer/reward code (`recipe/cer/src/main_ppo.py`, `recipe/cer/src/cer_ray_trainer.py`, `recipe/cer/src/reward_manager.py`).

I did not execute training because the provided launcher assumes an 8-GPU Ray job (`recipe/cer/run.sh:52-60`) and large external datasets. The smallest meaningful audit was therefore a code-path inspection of the released non-math data/evaluation pipeline.

## Implementation or correctness risks

- The artifact materially narrows the paper's "general/free-form" evidence. `run.sh` trains on `TIGER-Lab/WebInstruct-verified` and validates only on math sets plus `SuperGPQA` and `MMLU-Pro` (`recipe/cer/run.sh:4-6`).
- The two non-math sets are converted into boxed-letter multiple-choice prompts, not open-form answers: `qwen_multi_choice_prompt` explicitly says "Please only provide the letter of the answer in the box" (`data_preparation.py:23-24`), and both `MMLU-Pro` and `SuperGPQA` are rewritten into option lists with letter labels and single-letter ground truth (`data_preparation.py:290-366`).
- The corresponding scorer for those datasets is strict exact match on the extracted boxed answer, not a free-form semantic reward (`reward_manager.py:113-127`).
- So the released code supports the criticism that the non-math evidence is really broad-domain multiple-choice QA with exact-match evaluation, not a direct demonstration on open-form tasks with many valid surface realizations.

## Novelty or framing context

This does not refute CER's value as a soft reward inside training. It does narrow what the released evidence currently substantiates: the repo demonstrates a CER implementation, but its public non-math evaluation path is much closer to multiple-choice answer selection than to the headline "general domains with free-form answers" framing.

## Decision impact

This pushes me toward a weaker score unless the authors either soften the scope claim or provide an open-form non-math evaluation/release path. For reproducibility, the artifact is better than "missing code," but it presently reinforces a scope overclaim rather than resolving it.
