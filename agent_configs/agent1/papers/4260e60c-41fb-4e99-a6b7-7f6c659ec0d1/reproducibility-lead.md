# Reproducibility Lead Report

## Paper

- Paper ID: `4260e60c-41fb-4e99-a6b7-7f6c659ec0d1`
- Title: `Demystifying When Pruning Works via Representation Hierarchies`
- Role: Reproducibility Lead

## Claims Tested

1. The central empirical claim is that pruning often preserves non-generative task performance while substantially degrading generative performance.
2. The central mechanistic claim is that hidden-state and logit representations remain comparatively stable, but the softmax probability space amplifies pruning-induced logit perturbations; this then compounds during autoregressive generation.
3. The central artifact claim is that the released code is sufficient to support the paper's analysis and practical guidance.

Minimum reproduction target set before synthesis: at least one role should be able to recover the headline table/figure values or raw analysis metrics from released artifacts, and a second independent role should be able to verify the same claim from an independent route. Static paper inspection alone was not counted as reproduction.

## Evidence Gathered

Local checks used these sources:

- Paper TeX: `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/main.tex`
- Paper sections: `artifacts/sections/introduction.tex`, `artifacts/sections/method.tex`, `artifacts/sections/experiments.tex`, `artifacts/sections/appendix.tex`, `artifacts/sections/related_works.tex`
- Bibliography: `artifacts/references.bib`
- Author repo: `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations`, commit `ba1c25bd4d08c27319bf85d5a75b381fa70279d6`
- Koala discussion: one existing comment, `15f36ff9-c892-4d10-9b6a-d66d5c1d5b35`, limited to bibliography formatting.

Commands run:

```bash
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1 -maxdepth 3 -type f
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations -maxdepth 3 -type f
rg -n "Drop-8A|Drop-8M|GSM8K|HumanEval|e5-mistral|58\\.9|69\\.3|22\\.3" artifacts/sections -g "*.tex"
python --version
python -m py_compile representation-analysis/transition_layerwise_compare.py representation-analysis/compare_generation_metrics.py representation-analysis/compare_mcq_subspace_metrics.py representation-analysis/generation_forward_utils.py transition_metrics_logging.py intra-layer/main.py intra-layer/lib/data.py intra-layer/lib/prune.py
find . -maxdepth 5 -type f \( -name "*.json" -o -name "*.csv" -o -name "*.jsonl" -o -name "*.log" -o -name "*.txt" -o -name "*.pt" -o -name "*.pth" -o -name "*.npy" -o -name "*.pkl" \)
```

Observed environment:

- Python: `3.12.12`
- `torch`, `transformers`, `datasets`, and `accelerate` were not installed in the current environment.
- Static syntax compilation of the main analysis/pruning scripts succeeded.

## Role Assignments

- Independent Reproducer A: verify the headline empirical pattern and representation hierarchy from paper artifacts and lightweight code checks.
- Independent Reproducer B: independently test whether README and released scripts are sufficient to recreate table/figure metrics.
- Implementation Auditor: inspect code/repo completeness, seeds, configs, hardcoded paths, result artifacts, and paper-code correspondence.
- Correctness Specialist: check Taylor/KL math, KL direction, causal attribution, task framing, and statistical validity.
- Literature Specialist: compare novelty/framing against cited pruning and layer-dropping work, especially Wanda, SparseGPT, Gromov et al., ShortGPT, Layer Drop/LLM-Drop, and related compression baselines.

## Preliminary Synthesis

The paper text supports that the authors ran a broad comparison: Table `tab:e5-mistral-prune` reports E5-Mistral retrieval averages of 58.9 full, 53.4 Drop-8A, and 56.8 Drop-8M, and Mistral multiple-choice averages of 69.3 full, 69.8 Drop-8A, and 64.3 Drop-8M, while generative averages drop from 22.3 full to 13.2 Drop-8A and 0.8 Drop-8M. This is internally consistent with the claimed discrepancy, but these values are only paper/table values unless raw logs or scripts reproduce them.

The author repo contains plausible analysis scripts and figures, but not raw benchmark outputs, dropped/pruned checkpoint configs, numeric figure data, sampled C4 calibration indices, full benchmark commands, or result logs. README commands require placeholder paths such as `/path/to/dropped_results`, `/path/to/dense_model`, and `/path/to/pruned_model`, and the only data-like files found within five levels are small C4 demo/config JSON files inside the vendored inter-layer code.

The mechanistic analysis has a plausible local Taylor basis, but its acceptance relevance depends on empirical validation. The released artifacts do not allow more than one internal role to reproduce the central empirical result under the stated setup. The appropriate reproducibility classification is weak reproduction of the central empirical claim, with partial conceptual support from paper figures and source code.

## Decision Impact

This should materially lower confidence. The paper may still contain a useful explanatory framing, but the central empirical and mechanistic claims are not independently reproducible from the released package. A public Koala comment should foreground this as a decision-relevant artifact/reproducibility limitation rather than a mere packaging complaint.

## Remaining Uncertainty

The missing raw outputs might exist outside the submitted artifacts or be regenerable with large compute and private local paths, but that is not acceptable evidence under agent1's reproducibility standard. The current review should not give credit for unreleased checkpoints, hidden logs, or unstated evaluation commands.
