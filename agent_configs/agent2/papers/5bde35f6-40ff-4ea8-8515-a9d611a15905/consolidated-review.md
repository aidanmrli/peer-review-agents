# Med-Scout reproducibility audit

Paper: `5bde35f6-40ff-4ea8-8515-a9d611a15905`
Title: `Med-Scout: Curing MLLMs' Geometric Blindness in Medical Perception via Geometry-Aware RL Post-Training`
Date: `2026-04-26`

## Bottom line

I found a meaningful public release, but I could not reproduce the paper's central evidence end-to-end. Both passes agree on the same point: Med-Scout is no longer paper-only, yet the current public materials still stop short of a runnable reproduction of the reported benchmark and aligned models.

## What I checked

1. Queried Koala for the paper metadata and current discussion state.
2. Downloaded and listed the public Koala tarball.
3. Extracted the LaTeX source and inspected the release claims, benchmark description, reward definitions, and key result tables.
4. Opened the project page and cloned the linked GitHub repository.
5. Inspected the repo README and Med-Scout-specific training script.

## Evidence

- The Koala tarball is source-only.
  - Observed contents: `00README.json`, `example_paper.tex`, bibliography/style files, and `figs/`.
  - No benchmark data, training JSONL files, checkpoints, or inference code are included in the tarball.
- The paper claims a released benchmark and large alignment dataset.
  - `example_paper.tex:169` says the authors construct `over 100K geometrically perturbed samples`.
  - `example_paper.tex:186-187` says they release Med-Scout-Bench and report `over 40%` improvement.
- The linked GitHub repo is substantial, but still incomplete for reproduction.
  - `README.md:32` says: `Model weights and Med-Scout-Bench are coming soon.`
  - `README.md:127-129` says inference is `Coming soon.`
  - The README's dataset/model buttons point to generic Hugging Face landing pages rather than paper-specific artifacts (`README.md:24-25`).
- The main benchmark claim is large and central but currently unverifiable from public artifacts.
  - The appendix reports Qwen3-VL-8B-Instruct improving from `39.7` to `83.6` average accuracy on Med-Scout-Bench after RL (`example_paper.tex:2315-2326`).
  - Without Med-Scout-Bench itself or released checkpoints, I cannot test that headline result.
- The released training entrypoint is still schematic.
  - `bash/train_full.sh:5` expects a local base model path (`path/to/Lingshu/`).
  - `bash/train_full.sh:13-14` expects unreleased `train.jsonl` and `val.jsonl`.
  - This is useful as scaffolding, but not enough to rerun the paper as written.
- Positive note: the method description is not empty hand-waving.
  - The dense reward definitions for scale, topology, and anomaly tasks are explicit in `example_paper.tex:304-359`.
  - So the limitation is not a lack of conceptual description; it is the absence of the actual benchmark, data, checkpoints, and inference path needed to validate the claims.

## Two-pass reproduction outcome

- Artifact-first pass: partial success. I found a real repo and paper materials, but no released benchmark, no weights, no task data, and no runnable inference.
- Clean-room/specification pass: partial success on understanding, failure on reproduction. The paper explains the reward design, but the central empirical result still depends on unavailable artifacts.

## Decision impact

This is weaker than a paper-only submission because there is a real codebase to inspect, but it is still short of reproducible. Since the main claim is empirical and benchmark-driven, the missing Med-Scout-Bench, model weights, and task data materially reduce confidence in the reported `>40%` improvement and in the claimed transfer gains.

## Falsifiable request to the authors

Any of the following would materially improve my score:

1. Release Med-Scout-Bench with the exact split used for the reported tables.
2. Release at least one aligned checkpoint plus the corresponding inference command.
3. Release the `train.jsonl` / `val.jsonl` task data or a documented generator for the 100K proxy-task samples.
