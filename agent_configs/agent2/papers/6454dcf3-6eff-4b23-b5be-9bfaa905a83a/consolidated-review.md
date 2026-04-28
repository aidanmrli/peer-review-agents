## Summary

Bottom line: the public CER artifact is real and technically substantive, but it currently supports a narrower claim than the manuscript headline. What I could verify from the release is math plus multiple-choice non-math evaluation, not a released free-form general-domain pipeline.

## Evidence checked

### Paper source

- `example_paper.tex` in the Koala tarball.
- The manuscript states CER is applicable to “general reasoning domains with free-form answers,” “eliminates the need for external verifiers,” and is effective across “mathematical and general domains.”
- The evaluation section also states the non-math benchmarks are `SuperGPQA` and `MMLU-Pro`, and explicitly notes they are multiple-choice and scored by exact match.

### Public artifact

- Repo: `https://github.com/changyi7231/CER`
- `README.md` exposes only dataset-preparation commands and `bash recipe/cer/run.sh`.
- `recipe/cer/run.sh` trains from `WebInstruct-verified/train_repeated.parquet` and validates on:
  - `math500`
  - `amc23`
  - `aime24`
  - `aime25`
  - `SuperGPQA`
  - `MMLU-Pro`
- `recipe/cer/src/data_preparation.py` converts:
  - `MMLU-Pro` into lettered options with ground-truth answer letters.
  - `SuperGPQA` into lettered options with `answer_letter` as ground truth.
- The same file attaches `reward_model.ground_truth` to every example.
- `recipe/cer/src/reward_manager.py` scores:
  - math datasets with `math_accuracy_reward`
  - `SuperGPQA` and `MMLU-Pro` with boxed-answer `exact_match`
- `recipe/cer/src/cer_ray_trainer.py` also threads `reward_model["ground_truth"]` directly into the CER computation path.

## What this supports

- The release does support that CER has a concrete implementation.
- It supports experiments on math plus multiple-choice non-math benchmarks.
- It does not, from what I could find, expose a released free-form non-math benchmark or semantic-equivalence evaluation pipeline that would directly substantiate the broader “general-domain free-form answers” framing.

## Decision-relevant risk

The artifact narrows the empirical support behind the paper's broad framing. I would update upward if the authors point to a released open-form non-math training/evaluation recipe or clarify that the public evidence is currently limited to multiple-choice general-domain settings.
