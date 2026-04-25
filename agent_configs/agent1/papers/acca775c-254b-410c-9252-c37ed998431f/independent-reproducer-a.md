# Expert Threshold Routing - Independent Reproducer A

- Paper ID: `acca775c-254b-410c-9252-c37ed998431f`
- Title: `Expert Threshold Routing for Autoregressive Language Modeling with Dynamic Computation Allocation and Load Balancing`
- Assigned role: Independent Reproducer A
- Date: 2026-04-25

## Task scope

Start from the paper text and official repository, run the documented code path if feasible, and determine whether the reported ET improvement over TC-MoE can be recovered or at least sanity-checked.

## Claim attempted

The paper claims that ET routing scales to a 2.4B-parameter d20 model and achieves 0.067 lower cross entropy than TC-MoE, equivalent to 1.6x fewer tokens (`artifacts/v2.tex:138`, `artifacts/v2.tex:146-148`, `artifacts/v2.tex:304-318`, `artifacts/v2.tex:349-363`).

## Evidence examined

- `artifacts/v2.tex:138`
- `artifacts/v2.tex:304-318`
- `artifacts/v2.tex:321-363`
- `artifacts/v2.tex:848-854`
- `repos/Expert-Threshold-Routing/README.md:51-60`
- `repos/Expert-Threshold-Routing/README.md:62-103`
- `repos/Expert-Threshold-Routing/requirements.txt:1-23`
- `repos/Expert-Threshold-Routing/script/train.sh:84-117`

## Setup used

- Working directory: `papers/acca775c-254b-410c-9252-c37ed998431f/repos/Expert-Threshold-Routing`
- Repository commit: `534360cc08ae2d850c4d91646f5976381423a231`
- Local Python: `Python 3.12.12`
- No dependency installation was performed.

## Commands and observed outputs

```bash
python --version
```

Output:

```text
Python 3.12.12
```

```bash
python - <<'PY'
for m in ['torch','hydra','omegaconf']:
    try:
        mod=__import__(m)
        print(f'{m}: ok {getattr(mod, "__version__", "unknown")}')
    except Exception as e:
        print(f'{m}: {type(e).__name__}: {e}')
PY
```

Output:

```text
torch: ModuleNotFoundError: No module named 'torch'
hydra: ModuleNotFoundError: No module named 'hydra'
omegaconf: ModuleNotFoundError: No module named 'omegaconf'
```

```bash
MODEL_SIZE=tiny TRAINING_TOKENS=1 N_GPUS=1 ./script/train.sh --mlp et --g 2 --e 8
```

Output:

```text
Traceback (most recent call last):
  File ".../repos/Expert-Threshold-Routing/train.py", line 11, in <module>
    import hydra
ModuleNotFoundError: No module named 'hydra'
```

## Result

Blocked. I could not execute even the documented tiny ET training example because the environment lacks required dependencies. More importantly, the released artifacts do not include raw outputs that would allow a no-rerun verification of the reported d12/d20 table values.

## Reproducibility blockers

- The README says the release includes training code, configs, minimal scripts, CORE evaluation, and WandB logging support, but explicitly excludes benchmark suites and visualization modules (`README.md:51-60`).
- No checkpoints, WandB exports, run IDs, seed manifests, raw validation-loss logs, CORE result JSON/CSV files, or figure-generation scripts were present under the repository or TeX artifacts.
- The d20 claim requires training on 11.2B FineWeb-Edu tokens with 8x NVIDIA B200 180GB GPUs (`artifacts/v2.tex:304-309`, `artifacts/v2.tex:848-854`), which is not a reasonable reviewer rerun.

## Match status

Blocked for empirical reproduction. The documented command path is plausible, but the acceptance-relevant numbers are not reproducible from the released package in this environment.

## Confidence level

High that the released artifacts are insufficient for independent recovery of the headline result. Low about whether the method itself would reproduce if the full internal training logs and compute were available.

## Decision impact

Major negative for reproducibility. A paper centered on scaling curves and loss deltas should provide raw curves, exact run configs, and checkpoints or at least verifiable logs.
