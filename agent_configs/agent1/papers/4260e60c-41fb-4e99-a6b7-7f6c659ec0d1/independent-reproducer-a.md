# Independent Reproducer A Report

Paper: `4260e60c-41fb-4e99-a6b7-7f6c659ec0d1`, "Demystifying When Pruning Works via Representation Hierarchies"

Role: Independent Reproducer A. I used only the paper source artifacts and the mirrored author repository. I did not install dependencies or run model inference.

## Claim Tested

Central empirical claim: pruning is comparatively safe for non-generative tasks but can fail on generative tasks, and this is explained by a representation hierarchy where hidden states `h` and logits `z` remain relatively stable while the probability distribution `p = softmax(z / T)` shifts enough to destabilize autoregressive decoding.

## Sources Checked

- Paper source: `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/introduction.tex`, `method.tex`, `experiments.tex`, `appendix.tex`
- Paper figures: `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/figures/`
- Author repo mirror: `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations`
- Repo commit verified: `ba1c25bd4d08c27319bf85d5a75b381fa70279d6`
- Local role instruction: `skills/independent-reproducer-a.md`

## Commands Run

```bash
sed -n '1,220p' skills/independent-reproducer-a.md
git -C papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations rev-parse HEAD
git -C papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations status --short
nl -ba papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/{introduction,method,experiments,appendix}.tex
sed -n '1,240p' papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/README.md
python --version
sed -n '1,220p' papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/requirements.txt
python -m py_compile papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/transition_layerwise_compare.py \
  papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/compare_generation_metrics.py \
  papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/compare_mcq_subspace_metrics.py \
  papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/transition_metrics_logging.py
python papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/compare_generation_metrics.py --help
python papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/compare_mcq_subspace_metrics.py --help
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations -maxdepth 4 -type f -name 'config.json'
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations -maxdepth 4 -type d \( -name '*checkpoint*' -o -name '*results*' \)
awk 'BEGIN {printf("MCQ: %.1f %.1f; gen: %.1f %.1f; retrieval: %.1f %.1f\n", 69.8-69.3, 64.3-69.3, 13.2-22.3, 0.8-22.3, 53.4-58.9, 56.8-58.9)}'
```

Environment observed: `Python 3.12.12`. `torch` is not installed in this environment, so runtime entry points failed before argument parsing with `ModuleNotFoundError: No module named 'torch'`. The repository requires pinned `torch==2.7.1`, `transformers==4.52.4`, `accelerate==0.34.2`, and related packages.

## Reproduction Attempt and Outcome

The paper source clearly states the target claim. `introduction.tex` lines 32-39 say pruned models often retain non-generative performance but fail on generative tasks, and frame the mechanism as stable embedding/logit representations with amplified probability-space perturbations. `experiments.tex` lines 73-80 define the measured spaces `h`, `z = W h`, and `p = softmax(z / T)`.

I could statically verify the paper-table arithmetic for the Mistral layer-dropping result in `method.tex` lines 57-112. Non-generative multiple-choice average changes are small relative to generation: MCQ average is 69.3 full, 69.8 Drop-8A, and 64.3 Drop-8M, i.e. +0.5 and -5.0 points. Generative average is 22.3 full, 13.2 Drop-8A, and 0.8 Drop-8M, i.e. -9.1 and -21.5 points. Retrieval average is 58.9 full, 53.4 Drop-8A, and 56.8 Drop-8M, i.e. -5.5 and -2.1 points. This supports the reported non-generative/generative discrepancy within the paper's own source tables.

I also verified that the repository contains code paths intended to measure the representation hierarchy. `transition_metrics_logging.py` logs hidden similarity (`emb sim`), logit/head similarity (`head sim`, `Z_real(1-cos)`), probability similarity (`vocab sim`, `REAL(1-cos)`), and distributional shift (`REAL_KL`, `KL_estimate`). `compare_generation_metrics.py` computes per-step hidden/logit/probability cosine similarities, KL divergence, and variance-based estimates. `compare_mcq_subspace_metrics.py` compares global vocabulary behavior with option-token subspace behavior. These code paths match the claimed `h -> z -> p` analysis at the implementation-outline level.

However, I did not independently reproduce the numeric figures or benchmark results. The scripts compile syntactically, but they cannot run in the current environment because `torch` is absent. Even with dependencies, the repository mirror does not include the exact dropped checkpoint configs, pruned checkpoints, raw benchmark outputs, cosine logs, or figure-generation data needed to regenerate the paper figures under the stated review constraints. `find` found no checked-in `config.json` checkpoint files or result directories corresponding to the README command placeholders.

Outcome: partial static reproduction only. The claim is consistent with the paper tables/figures and the repository's intended metrics, but the central empirical result is not independently reproduced from executable artifacts in this pass.

## Blockers and Reproducibility Issues

- Missing runtime dependencies: both lightweight `--help` checks failed on `import torch`.
- Missing result artifacts: no raw logs, benchmark JSON/output files, dropped-model configs, pruned checkpoints, or saved analysis tensors were present in the allowed repository mirror.
- Placeholder paths: README commands require `/path/to/dropped_results`, `/path/to/dense_model`, and `/path/to/pruned_model`; no concrete paper-aligned paths are supplied.
- Calibration mismatch: appendix implementation says pruning masks use 128 randomly sampled C4 sequences, but `inter-layer/scripts/dropping/layer_drop.sh` sets `n_calibration_samples=256`.
- Default script mismatch: `intra-layer/scripts/prune.sh` defaults to Mistral and placeholder model roots, while the main intra-layer figure is described for Qwen-2.5-7B-Instruct.
- Prompt coverage is narrow: representation-analysis scripts hard-code one or a few toy prompts. `compare_generation_metrics.py` uses the "John has twice as many books" prompt; the main generation-collapse table uses the Natalia clips prompt; `compare_mcq_subspace_metrics.py` uses lowercase option tokens `[" a", " b", " c", " d"]` rather than the uppercase A/B/C/D formulation described in the paper source.
- No benchmark reproduction path was runnable under constraints. The inter-layer `benchmark_lm_eval.sh` downloads/clones models and runs expensive evaluation over multiple tasks; this exceeds the lightweight-command limit and requires external dependencies and model artifacts.

## Acceptance Consequence

The central claim is plausible from the provided paper source and internally consistent with the repository's metric definitions, but it remains only weakly to partially reproducible from the submitted artifacts under this review pass. I would materially mark down reproducibility unless the authors release exact checkpoint configs, pruning masks or pruned model identifiers, raw evaluation outputs, cosine/KL logs, seeds, and figure-generation scripts/data sufficient to regenerate the reported non-generative/generative performance gap and the `h/z` stability versus `p` shift.
