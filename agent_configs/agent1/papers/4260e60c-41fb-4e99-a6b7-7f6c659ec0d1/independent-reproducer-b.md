# Independent Reproducer B Report

Paper: 4260e60c-41fb-4e99-a6b7-7f6c659ec0d1  
Title: Demystifying When Pruning Works via Representation Hierarchies  
Role: Independent Reproducer B, repo/README reproduction route

## Claim Tested

I tested whether the released author repository and README commands are sufficient to recreate the paper's central empirical artifacts, or at least the analysis metrics behind them: pruning preserves non-generative behavior more than generative behavior, and representation-hierarchy metrics show relatively stable embedding/logit spaces but larger probability-space deviation under pruning.

The concrete targets were paper Figure 1-8/Table examples as exposed in `artifacts/sections/experiments.tex`, especially:

- Figure `intra_layer_pruning_gng.pdf`: HellaSwag vs GSM8K under Wanda sparsity.
- Table `tab:case_study`: Qwen-2.5-7B-Instruct outputs under Drop-4/8 attention or MLP.
- Figure `representation_sim`: layerwise embedding/logit/probability similarity.
- Figure `est`: ground-truth vs theoretical estimates for 1 - cosine and KL.
- Figure `final_sim` and `local_global`: generation-step and MCQ subspace comparisons.

## Sources Checked

- Paper/source artifacts: `artifacts/main.tex`, `artifacts/source.tar.gz`, `artifacts/sections/experiments.tex`, `artifacts/sections/appendix.tex`.
- Author repository: `repos/Pruning-on-Representations` at local Git commit `ba1c25b` (`Update index.html`, 2026-04-12), remote `https://github.com/CASE-Lab-UMD/Pruning-on-Representations`.
- README and environment: `README.md`, `requirements.txt`.
- Analysis code: `representation-analysis/transition_layerwise_compare.py`, `compare_generation_metrics.py`, `compare_mcq_subspace_metrics.py`, `generation_forward_utils.py`, `transition_metrics_logging.py`.
- Pruning scripts: `intra-layer/scripts/prune.sh`, `inter-layer/scripts/dropping/*.sh`, `inter-layer/scripts/benchmark/benchmark_lm_eval.sh`, `inter-layer/scripts/quantization/*.sh`.
- Custom model code: `modeling_qwen.py`.

I did not read Reproducer A's report.

## Commands Run

No dependencies were installed and no model inference was run.

```bash
P=papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,220p' skills/independent-reproducer-b.md
find "$P" -maxdepth 2 -type f ! -name 'independent-reproducer-a.md' -print
tar -tzf "$P/artifacts/source.tar.gz" | sort
rg -n "(Figure|Table|prun|generat|non-generative|embedding|logit|probability|KL|cosine)" "$P/artifacts" -g '*.tex'
sed -n '1,560p' "$P/repos/Pruning-on-Representations/README.md"
sed -n '1,560p' "$P/repos/Pruning-on-Representations/representation-analysis/transition_layerwise_compare.py"
sed -n '1,560p' "$P/repos/Pruning-on-Representations/representation-analysis/compare_generation_metrics.py"
sed -n '1,520p' "$P/repos/Pruning-on-Representations/representation-analysis/compare_mcq_subspace_metrics.py"
sed -n '1,560p' "$P/repos/Pruning-on-Representations/representation-analysis/generation_forward_utils.py"
sed -n '1,280p' "$P/repos/Pruning-on-Representations/transition_metrics_logging.py"
python -m py_compile "$P"/repos/Pruning-on-Representations/representation-analysis/*.py "$P"/repos/Pruning-on-Representations/transition_metrics_logging.py "$P"/repos/Pruning-on-Representations/intra-layer/main.py "$P"/repos/Pruning-on-Representations/intra-layer/lib/*.py
bash -n "$P/repos/Pruning-on-Representations/intra-layer/scripts/prune.sh"
bash -n "$P"/repos/Pruning-on-Representations/inter-layer/scripts/dropping/*.sh "$P"/repos/Pruning-on-Representations/inter-layer/scripts/benchmark/*.sh "$P"/repos/Pruning-on-Representations/inter-layer/scripts/quantization/*.sh
(cd "$P/repos/Pruning-on-Representations" && python transition_layerwise_compare.py --help)
(cd "$P/repos/Pruning-on-Representations/representation-analysis" && python transition_layerwise_compare.py --help)
(cd "$P/repos/Pruning-on-Representations/representation-analysis" && python compare_generation_metrics.py --help)
(cd "$P/repos/Pruning-on-Representations/representation-analysis" && python compare_mcq_subspace_metrics.py --help)
find "$P/repos/Pruning-on-Representations" -type f \( -name '*.json' -o -name '*.csv' -o -name '*.log' -o -name '*.pt' -o -name '*.safetensors' \) -not -path '*/.git/*' -print
find "$P/repos/Pruning-on-Representations" -maxdepth 3 -type d \( -name 'cosine_logs' -o -name '*checkpoint*' -o -name '*result*' -o -name '*dropped*' -o -name '*pruned*' \) -print
```

## Reproduction Attempt and Outcome

Outcome: blocked for figure/table reproduction; partial for static script audit.

The repository contains pre-rendered figures in `figs/` and `docs/figs/`, but I found no saved metric logs, table outputs, model checkpoints, pruned model directories, dropped-model configs, or plotting scripts sufficient to regenerate the paper figures from raw or intermediate data. The only JSON/large data-like files found were C4 demo/train/validation files under the inter-layer LLaMA-Factory-style tree. There were no `cosine_logs`, result directories, checkpoint directories, `.pt` metric caches, or benchmark output JSONs in the released repo.

The README commands are not copy-paste reproducible from the repository root. For example:

```bash
cd repos/Pruning-on-Representations
python transition_layerwise_compare.py --help
```

failed with:

```text
python: can't open file '.../Pruning-on-Representations/transition_layerwise_compare.py': [Errno 2] No such file or directory
```

Running from `representation-analysis/` reaches imports, but my local environment has no `torch`; per the role instructions I did not install dependencies:

```text
ModuleNotFoundError: No module named 'torch'
```

The syntax-only checks did pass: `python -m py_compile` over the main analysis/pruning Python files returned no syntax errors, and `bash -n` passed for the intra-layer and inter-layer shell scripts.

The stronger reproducibility blocker is not my local missing `torch`; it is artifact incompleteness. The README examples require reviewer-supplied paths such as `/path/to/dropped_results`, `/path/to/dense_model`, and `/path/to/pruned_model`. The repo does not provide those directories, the exact dropped-layer/pruned checkpoint artifacts, or commands that produce the paper's Qwen/Mistral/LLaMA/Qwen3 result sets end to end.

There is also a code-path concern for the README's dropped-mode analysis. `generation_forward_utils.apply_drop_masks` only sets `layer.drop_attn` and `layer.drop_mlp` if those attributes already exist. The README example uses `--model_name Qwen/Qwen2.5-7B-Instruct`, which should load a standard Hugging Face Qwen model, not necessarily the custom dropped-layer implementation. The repo includes `modeling_qwen.py`, but it is not wired into the README command, and the file imports `.configuration_qwen2`, which I could not find in the released repo. Static inspection therefore does not establish that the dropped-mode README command actually applies the advertised layer drops when run as documented.

## Blockers

- No raw metrics, benchmark outputs, cosine/KL logs, dropped-model configs, pruned checkpoints, or figure-generation notebooks/scripts are released.
- README commands require unspecified external artifacts: dropped results roots, dense local model paths, pruned model paths, and large HF models.
- The analysis scripts use one or two hard-coded prompts, while the paper claims averages/ranges over multiple prompts, benchmarks, models, sparsity modes, and generation steps.
- Benchmark reproduction for HellaSwag, MMLU, GSM8K, HumanEval, BEIR, etc. is not exposed as a paper-aligned pipeline in the top-level README. The inter-layer benchmark script is a template that clones Mistral and expects local `*_drop*` configs, but those configs are absent.
- The environment is heavy (`torch==2.7.1` CUDA 12.8, transformers, accelerate, datasets), and reproduction would require GPU-scale model loading. This is expected for full reproduction, but it means the released repo must provide exact intermediate artifacts to support lightweight verification; it does not.

## Acceptance Consequence

This is a weak reproducibility result for the core empirical claim. I can verify that the authors released plausible code skeletons for computing cosine, KL, and second-order estimates, and the syntax checks do not reveal immediate parse failures. I cannot independently recreate the paper's figures, tables, or even representative logged metrics from the released repo plus README without unreleased checkpoints/results and substantial undocumented setup.

The acceptance impact is materially negative under a reproducibility-first rubric. The paper's central explanation may be correct, but the public artifact currently supports only inspection of pre-rendered figures and code structure, not independent recovery of the reported empirical evidence.

## Agreement With Reproducer A

Not assessed in this independent pass. I intentionally did not read Reproducer A's report; comparison should be done after both role reports are complete.
