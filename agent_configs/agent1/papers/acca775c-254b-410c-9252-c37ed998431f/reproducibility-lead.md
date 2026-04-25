# Expert Threshold Routing - Reproducibility Lead

- Paper ID: `acca775c-254b-410c-9252-c37ed998431f`
- Title: `Expert Threshold Routing for Autoregressive Language Modeling with Dynamic Computation Allocation and Load Balancing`
- Assigned role: Reproducibility Lead
- Date: 2026-04-25

## Task scope

Coordinate the internal review of the paper's core empirical and methodological claims, then decide whether at least two independent internal roles reproduced the central acceptance-relevant result.

## Claims tested

1. ET routing is a causal approximation to EC routing via an EMA cutoff: each expert accepts tokens with router score above a learned threshold.
2. At the reported d20 scale, ET achieves 0.067 lower validation cross entropy than TC-MoE and reaches the same loss with 1.6x fewer tokens.
3. ET preserves near-perfect load balancing and small train-inference mismatch without auxiliary losses.
4. The released paper/code artifacts are sufficient for an independent reviewer to reproduce or strongly audit the main result.

## Evidence examined

- Paper source: `artifacts/v2.tex`
- Bibliography: `artifacts/example_paper.bib`
- Author repository: `repos/Expert-Threshold-Routing`, commit `534360cc08ae2d850c4d91646f5976381423a231`
- Key code paths:
  - `README.md`
  - `configs/config.yaml`
  - `configs/mlp/et.yaml`
  - `configs/mlp/ec.yaml`
  - `configs/training/standard.yaml`
  - `src/models/model_base.py`
  - `src/models/expert_threshold_choice.py`
  - `src/models/engines/common.py`
  - `train.py`
  - `eval_core.py`
  - `script/train.sh`
  - `script/download_data.sh`

## Role findings

- Independent Reproducer A attempted the documented tiny ET command and was blocked before execution because the local Python environment lacks `hydra`, `omegaconf`, and `torch`. This is not by itself a paper failure, but no released logs, checkpoints, seeds, or result tables exist to verify the d12/d20 claims without rerunning 10B-11.2B-token training.
- Independent Reproducer B used a static reconstruction path. The ET mechanism exists in code, but the experiment description in `artifacts/v2.tex:304-309` and `artifacts/v2.tex:715-761` states `G=1,E=16` with a shared expert, while released configs use `G=2,E=8` (`configs/mlp/et.yaml:4-19`, `configs/mlp/ec.yaml:4-18`) and the model asserts shared-expert `granularity >= 2` (`src/models/model_base.py:123-130`). This prevents exact reconstruction of the stated configuration.
- The Implementation Auditor found a plausible core implementation of top-k/threshold routing, but no raw WandB exports, checkpoints, paper-table run manifests, figure-generation code, or CORE benchmark result files.
- The Correctness Specialist found a major paper-code/configuration inconsistency and an unsupported leap from plotted/claimed training curves to the "1.6x fewer tokens" conclusion because the source artifacts only include final figures, not the underlying points/interpolation.
- The Literature Specialist found that novelty is best framed as a specific EMA-threshold causalization of EC for autoregressive language modeling, not as the first dynamic-compute or auxiliary-loss-free MoE routing method. The paper cites many relevant works but still overstates the load-balancing and baseline strength of the d20 comparison.

## Commands and environment details

Commands run from `repos/Expert-Threshold-Routing`:

```bash
python --version
python - <<'PY'
for m in ['torch','hydra','omegaconf']:
    try:
        mod=__import__(m)
        print(f'{m}: ok {getattr(mod, "__version__", "unknown")}')
    except Exception as e:
        print(f'{m}: {type(e).__name__}: {e}')
PY
MODEL_SIZE=tiny TRAINING_TOKENS=1 N_GPUS=1 ./script/train.sh --mlp et --g 2 --e 8
git rev-parse HEAD
```

Observed:

```text
Python 3.12.12
torch: ModuleNotFoundError: No module named 'torch'
hydra: ModuleNotFoundError: No module named 'hydra'
omegaconf: ModuleNotFoundError: No module named 'omegaconf'
ModuleNotFoundError: No module named 'hydra'
534360cc08ae2d850c4d91646f5976381423a231
```

## Reproduction outcome

The central empirical claim was not reproduced by two independent internal roles.

- Reproducer A: blocked at execution and no released results artifacts to compare against.
- Reproducer B: partially validated the method logic statically but found an exact-configuration mismatch that prevents faithful reproduction of the stated experiment.

This is weak reproducibility. The method may be real and implemented, but the acceptance-relevant numbers are not independently recoverable from the released evidence.

## Decision impact

Major negative. The paper's main contribution is empirical: ET's d20 CE/CORE advantage over TC and EC-style causalization. Without raw runs or exact configs, the evidence supports only a partial implementation audit, not independent reproduction of the key result. Recommended score impact: weak reject unless stronger released logs/configs/checkpoints are provided.

## Limitations

I did not install CUDA PyTorch or run multi-GPU training. That is acceptable for this review because the paper's headline claim requires 8x B200 training over 10B-11.2B tokens (`artifacts/v2.tex:848-854`), and the artifact package should provide enough recorded evidence for reviewers who cannot rerun that scale.

## Confidence level

High for the reproducibility assessment and paper-code mismatch. Moderate for the expected effect on final quality because the underlying method implementation is plausible.
