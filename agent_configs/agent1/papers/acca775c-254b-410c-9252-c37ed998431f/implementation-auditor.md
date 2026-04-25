# Expert Threshold Routing - Implementation Auditor

- Paper ID: `acca775c-254b-410c-9252-c37ed998431f`
- Title: `Expert Threshold Routing for Autoregressive Language Modeling with Dynamic Computation Allocation and Load Balancing`
- Assigned role: Implementation Auditor
- Date: 2026-04-25

## Task scope

Audit the author-linked repository and paper artifacts for paper-code consistency, reproducibility support, missing dependencies, missing result artifacts, and whether the implementation computes the reported quantities.

## Artifact inventory

- Paper TeX: `artifacts/v2.tex`
- Bibliography: `artifacts/example_paper.bib`
- Paper figures: `artifacts/figures_v2/...`
- PDF: `artifacts/paper.pdf`
- Author repository: `repos/Expert-Threshold-Routing`
- Repository commit: `534360cc08ae2d850c4d91646f5976381423a231`

No raw logs, checkpoints, WandB exports, run manifests, metric CSV/JSON files, or figure-generation scripts were found. A search for result/checkpoint/log-like files returned only repository support files and static images/PDFs.

## Code paths inspected

- `README.md:51-60`
- `README.md:62-103`
- `requirements.txt:1-23`
- `configs/config.yaml:1-53`
- `configs/training/standard.yaml:1-10`
- `configs/mlp/et.yaml:1-22`
- `configs/mlp/ec.yaml:1-18`
- `configs/model_size/d20.yaml:1-10`
- `src/models/model_base.py:31-160`
- `src/models/expert_threshold_choice.py:29-114`
- `src/models/engines/common.py:13-120`
- `src/models/engines/common.py:190-272`
- `train.py:33-60`
- `train.py:181-299`
- `eval_core.py:52-93`
- `script/train.sh:1-117`
- `script/download_data.sh:1-16`

## Paper-to-code matches

- The repository contains a real ET/EC implementation path. `ExpertThresholdChoiceMLP` is a unified EC/ET wrapper (`src/models/expert_threshold_choice.py:29-30`).
- Training uses top-k routing before switching to threshold routing at the warmup step (`train.py:181-191`).
- The engine accumulates top-k cutoffs under top-k routing and uses EMA thresholds for threshold routing (`src/models/engines/common.py:43-53`, `src/models/engines/common.py:66-120`, `src/models/engines/common.py:251-272`).
- The d20 config provides the paper's reported depth/embedding/head values (`configs/model_size/d20.yaml:4-8`; compare `artifacts/v2.tex:739-745`).
- The default standard training config uses the reported 524,288-token batch and 10B-token d12 budget (`configs/training/standard.yaml:1-10`; compare `artifacts/v2.tex:781-782`).

## Paper-to-code discrepancies

1. **Experiment config mismatch.** The paper says MoE variants use `G=1,E=16` plus a shared expert (`artifacts/v2.tex:307`, `artifacts/v2.tex:717`, `artifacts/v2.tex:761`). The released ET/EC configs use `G=2,E=8` (`configs/mlp/et.yaml:9-10`, `configs/mlp/ec.yaml:9-10`). The model rejects shared expert `G<2` (`src/models/model_base.py:123-130`).
2. **Target-rate formula depends on the released setting.** With shared experts, `_compute_k_target` uses `(g-1)/(g*e)` (`src/models/engines/common.py:13-21`). The paper-stated `G=1,E=16` would route zero target tokens by this code, while the released `G=2,E=8` yields one routed expert per token on average.
3. **D20 token budget is not a config default.** `configs/training/standard.yaml:5` defaults to 10B tokens, while the paper reports 11.2B for d20 (`artifacts/v2.tex:309`, `artifacts/v2.tex:781-782`). This can be overridden via `script/train.sh:84-95`, but the exact d20 command/run manifest is absent.
4. **CORE evaluation requires checkpoints.** `eval_core.py:57-60` requires `eval.core_checkpoint_path`, but no checkpoints are released.
5. **Data script contains a hard-coded author path.** `script/download_data.sh:12` defaults to `/data2/hanchi/miniconda3/envs/nanochat/bin/python`, requiring manual override for other systems.

## Reproducibility blockers

- No recorded evidence for the reported table values or curves.
- No seeds per run beyond the default config seed.
- No exact Hydra override list for each reported baseline.
- No WandB run URLs or exports.
- No checkpoints for CE/CORE re-evaluation.
- No CORE benchmark suite files beyond wrapper code; README says benchmark suites are intentionally excluded (`README.md:51-60`).
- Headline compute requires 8x NVIDIA B200 180GB GPUs (`artifacts/v2.tex:848-854`).

## Severity for acceptance

Major. The method code is a meaningful positive artifact, but the reported empirical comparison is the acceptance case. The released artifacts are insufficient for reproducing the headline result, and the paper-code configuration mismatch weakens confidence that the released repository corresponds exactly to the paper experiments.

## Confidence level

High for artifact inventory, missing result files, and config mismatch. Moderate for implementation behavior because dependency absence prevented execution.
