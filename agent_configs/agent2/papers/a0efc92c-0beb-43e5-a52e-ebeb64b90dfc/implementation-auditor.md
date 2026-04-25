# Implementation Auditor Report

Paper: `a0efc92c-0beb-43e5-a52e-ebeb64b90dfc`  
Title: `Sequence Diffusion Model for Temporal Link Prediction in Continuous-Time Dynamic Graph`  
Role: Implementation Auditor  
Date: 2026-04-25  

## Claim Being Tested

The artifact should support the paper's central implementation claim: SDG is a trainable sequence-level diffusion model for temporal link prediction, with learnable node embeddings, causal sequence encoding, DDPM-style noising and reverse denoising over destination sequences, a cross-attention denoising decoder, cosine diffusion reconstruction loss, BCE/BPR ranking losses, tuned hyperparameters, and evaluation pipelines producing the reported MRR/HR@10, ablation, robustness, runtime, memory, and AP/ROC-AUC tables.

## Artifact Inventory

- `artifacts/example_paper.tex`, `artifacts/paper.pdf`, `artifacts/source.tar.gz`, and figure PDFs are present. `source.tar.gz` contains only paper source assets: `00README.json`, `example_paper.tex`, bibliography/style files, and figure PDFs. It does not contain implementation code.
- `artifacts/DyGLib` is a clean shallow clone of `https://github.com/yule-BUAA/DyGLib.git` at `3aacc36b94b8d2d8293d70a74fdf6d39089b4163` (`update paper bibtex`, 2023-09-28).
- `artifacts/TGB-Seq` is a clean shallow clone of `https://github.com/TGB-Seq/TGB-Seq.git` at `c1ee801ea4301c2f944f5b3da10cbc5310fa4f68` (`fix numpy package version issue`, 2025-12-21).
- No SDG-specific repository, branch, source package, checkpoints, logs, result JSONs, shell scripts, YAML configs, table-generation scripts, or saved model artifacts are present.
- The only nontrivial data files found locally are DyGLib's included `myket` sample/preprocessed files and TGB-Seq's toy counter CSV. The large paper datasets and TGB-Seq processed negative samples are not included.

## Code Paths Inspected

- Paper source:
  - `artifacts/example_paper.tex`
  - `artifacts/source.tar.gz`
- DyGLib:
  - `artifacts/DyGLib/train_link_prediction.py`
  - `artifacts/DyGLib/evaluate_link_prediction.py`
  - `artifacts/DyGLib/evaluate_models_utils.py`
  - `artifacts/DyGLib/utils/load_configs.py`
  - `artifacts/DyGLib/utils/metrics.py`
  - `artifacts/DyGLib/models/*`
  - `artifacts/DyGLib/README.md`
  - `artifacts/DyGLib/requirements.txt`
- TGB-Seq:
  - `artifacts/TGB-Seq/examples/train_link_prediction.py`
  - `artifacts/TGB-Seq/examples/evaluate_models_utils.py`
  - `artifacts/TGB-Seq/examples/evaluate_models_utils_mrr.py`
  - `artifacts/TGB-Seq/examples/utils/load_configs.py`
  - `artifacts/TGB-Seq/tgb_seq/LinkPred/evaluator.py`
  - `artifacts/TGB-Seq/tgb_seq/datasets/preprocess.py`
  - `artifacts/TGB-Seq/plots/*.py`
  - `artifacts/TGB-Seq/README.md`
  - `artifacts/TGB-Seq/pyproject.toml`

## Paper-to-Code Matches

- The paper states that DyGLib and TGB-Seq are used for baseline reproduction. Those two upstream repositories are present.
- DyGLib contains the expected baseline dynamic graph models listed in its own README: JODIE, DyRep, TGAT, TGN, CAWN, EdgeBank, TCL, GraphMixer, and DyGFormer.
- TGB-Seq contains a DyGLib-based example pipeline for TGB-Seq datasets, including multi-negative MRR evaluation. In `examples/evaluate_models_utils_mrr.py`, predefined TGB-Seq test negatives can be used when `evaluate_data.neg_samples is not None`.
- The TGB-Seq evaluator computes MRR from positive and negative scores via rank comparison. This partially matches the paper's MRR reporting for TGB-Seq-style ranking.

## Paper-to-Code Discrepancies

### 1. No SDG implementation is present

The paper describes SDG as a new model with sequence diffusion and cross-attention denoising. Relevant paper locations include:

- Abstract and contributions: `example_paper.tex` lines 105-123.
- Sequence diffusion model: lines 206-233.
- Denoising network: lines 235-247.
- Cosine diffusion loss and combined objective: lines 249-292.
- Inference/training algorithms: lines 998-1033.

I searched both repositories for SDG and diffusion-specific terms:

```bash
rg -n -g '!*.svg' -g '!*.pdf' -g '!*.bib' -g '!poetry.lock' \
  "SDG|Sequence Diffusion|Diffusion|diffusion|denois|lambda_diff|lambda_inter|cosine|BPR|Bayesian|repeat[-_ ]?time|CRAFT|noise schedule|alpha_bar|beta_|sqrt_alphas|timesteps|ddpm|ddim" \
  artifacts/DyGLib artifacts/TGB-Seq
```

This returned no matches. There is no `SDG` class, no diffusion scheduler, no denoising decoder, no target-sequence construction, no `lambda_diff`, no `lambda_inter`, no cosine diffusion loss, no BPR implementation for SDG, no repeat-time encoding implementation for SDG, and no inference loop over diffusion steps.

### 2. The training scripts cannot instantiate SDG

DyGLib `utils/load_configs.py` restricts `--model_name` to:

```text
JODIE, DyRep, TGAT, TGN, CAWN, EdgeBank, TCL, GraphMixer, DyGFormer
```

TGB-Seq `examples/utils/load_configs.py` has the same model choices. The corresponding training scripts instantiate only these baseline backbones and otherwise raise `ValueError("Wrong value for model_name ...")`. Running either pipeline with `--model_name SDG` would fail at argument parsing or model creation.

### 3. CRAFT baseline support is absent from the supplied code

The paper compares against CRAFT and states that CRAFT was handled specially by disabling shuffle-based training. I found no CRAFT model, config, or training branch in either supplied clone. This means the reported CRAFT baseline values and the claimed CRAFT protocol modification are not reproducible from the provided artifacts.

### 4. The reported HR@10 metric is not implemented in the inspected TGB-Seq evaluation path

The paper reports both MRR and HR@10 for ranking evaluation. The inspected TGB-Seq evaluator returns only an MRR list:

```python
optimistic_rank = (y_pred_neg > y_pred_pos).sum(axis=1)
pessimistic_rank = (y_pred_neg >= y_pred_pos).sum(axis=1)
ranking_list = 0.5 * (optimistic_rank + pessimistic_rank) + 1
mrr_list = 1./ranking_list.astype(np.float32)
return mrr_list
```

`examples/evaluate_models_utils_mrr.py` then returns only `{'mrr': np.mean(evaluate_metrics)}`. I did not find HR@10 computation in this evaluation path, so the paper's HR@10 tables cannot be regenerated from the supplied TGB-Seq example code.

### 5. Paper hyperparameters are not encoded as runnable configs

The paper reports SDG-specific optimal settings for each dataset: `L`, `K`, `lambda_inter`, `lambda_diff`, `N_layers`, batch size, embedding size, and task loss. None of these SDG-specific parameters are exposed in DyGLib or TGB-Seq configs. The available parser options cover baseline parameters such as `num_neighbors`, `num_layers`, `patch_size`, and `max_input_sequence_length`, but not diffusion steps, diffusion loss weight, intermediate-position loss weight, noise schedule, sequence target construction, or SDG embedding size.

### 6. Table, ablation, robustness, runtime, and memory results are not reproducible from artifacts

The paper reports:

- Main MRR/HR@10 tables for seen and unseen datasets.
- Ablations for `w/o Seq`, `w/o Diff`, `MSE`, and `MLP`.
- Hyperparameter sensitivity for diffusion steps and loss weights.
- Robustness under injected edge/timestamp noise.
- Runtime and memory comparisons.
- AP/ROC-AUC appendix results.

No result files, experiment logs, checkpoints, command scripts, aggregation notebooks, or plotting/table scripts for these SDG paper results are present. The only plot scripts under `TGB-Seq/plots` are benchmark-era scripts for EdgeBank/GraphMixer/DyGFormer/SGNN-HN or DyGLib baselines; they do not contain SDG and do not match the paper's SDG tables or figures.

### 7. Environment specification is incomplete and partly inconsistent

The paper states Python 3.11, PyTorch 2.0.1, and CUDA 11.8. DyGLib only specifies `torch>=1.8.1` with unpinned `numpy`, `pandas`, `tqdm`, and `tabulate`. TGB-Seq requires `torch>=2.3.0` and `numpy>=2.0`, which is inconsistent with the paper's stated PyTorch 2.0.1 environment. There is no lockfile for the SDG code because the SDG code is absent.

## Reproducibility Blockers

- Critical: The SDG source code is absent. The central model cannot be trained, evaluated, or inspected.
- Critical: No diffusion modules are present: no noising schedule, reverse diffusion loop, cross-attention denoiser, cosine reconstruction loss, or SDG inference code.
- Critical: No SDG configs or hyperparameter files are present for the 10 datasets.
- Critical: No checkpoints, logs, saved results, or raw command records are present for the paper's reported values.
- Major: The supplied baseline pipelines do not include CRAFT, although CRAFT is a major comparator in the paper.
- Major: The inspected TGB-Seq ranking evaluator returns MRR only, while the paper reports both MRR and HR@10.
- Major: TGB-Seq datasets and predefined negative-sample files are not included locally, so even baseline TGB-Seq runs require external data acquisition.
- Major: Paper figure/table generation scripts for SDG are absent. The included TGB-Seq plotting scripts are unrelated to the SDG paper results.
- Major: The environment is not reproducibly pinned and is inconsistent across the paper, DyGLib, and TGB-Seq artifacts.

## Commands and Checks Run

Key commands used during the audit:

```bash
git status --short
git -C artifacts/DyGLib rev-parse HEAD
git -C artifacts/DyGLib status --short
git -C artifacts/DyGLib log --oneline -5
git -C artifacts/TGB-Seq rev-parse HEAD
git -C artifacts/TGB-Seq status --short
git -C artifacts/TGB-Seq log --oneline -5
tar -tzf artifacts/source.tar.gz
find artifacts -path '*/.git' -prune -o \( -iname '*sdg*' -o -iname '*diff*' -o -iname '*denois*' -o -iname '*checkpoint*' -o -iname '*ckpt*' -o -iname '*log*' -o -iname '*config*' -o -iname '*result*' -o -iname '*table*' \) -print
find artifacts -path '*/.git' -prune -o -type d \( -name logs -o -name saved_models -o -name saved_results -o -name checkpoints -o -name configs -o -name results -o -name runs \) -print
find artifacts -path '*/.git' -prune -o -type f \( -name '*.json' -o -name '*.pt' -o -name '*.pth' -o -name '*.ckpt' -o -name '*.log' -o -name '*.csv' -o -name '*.npy' -o -name '*.npz' -o -name '*.pkl' -o -name '*.yaml' -o -name '*.yml' -o -name '*.sh' \) -print
rg -n "model_name|choices=|SDG|Diff|checkpoint|save|load_best|MRR|HR|negative|num_epochs|batch_size|learning_rate|optimizer|early" artifacts/DyGLib artifacts/TGB-Seq
rg -n -g '!*.svg' -g '!*.pdf' -g '!*.bib' -g '!poetry.lock' "SDG|Sequence Diffusion|Diffusion|diffusion|denois|lambda_diff|lambda_inter|cosine|BPR|Bayesian|repeat[-_ ]?time|CRAFT|noise schedule|alpha_bar|beta_|sqrt_alphas|timesteps|ddpm|ddim" artifacts/DyGLib artifacts/TGB-Seq
```

No random seeds or GPU experiments were run because there is no SDG implementation to execute. The relevant seed-bearing baseline scripts use default seeds internally, but they do not instantiate the submitted method.

## Reproduction Outcome

Implementation Auditor outcome: weak to absent reproducibility for the central implementation claim.

The supplied artifacts permit inspection of upstream DyGLib and TGB-Seq baseline code, but they do not permit an independent auditor to reproduce SDG. The central SDG results are therefore not independently reproducible from the provided code, configs, checkpoints, or scripts. This role did not execute Independent Reproducer A or B; their reports should separately record whether any non-code derivation or external reconstruction attempt can validate the paper's claims.

## Severity for Acceptance Decision

Severity: critical.

The implementation evidence does not support the paper's core empirical claims. The paper's acceptance case depends on SDG outperforming baselines through a specific sequence diffusion architecture, but the artifacts do not contain that architecture or the experimental machinery needed to regenerate the reported tables. This should impose a substantial negative score impact unless the authors provide a complete SDG implementation, exact configs, command scripts, checkpoints or logs, result aggregation scripts, and a reproducible HR@10/MRR evaluation pipeline before decision.
