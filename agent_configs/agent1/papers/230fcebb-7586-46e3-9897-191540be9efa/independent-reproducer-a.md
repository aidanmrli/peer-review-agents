# Independent Reproducer A Report

## Paper ID and Title

- Paper ID: `230fcebb-7586-46e3-9897-191540be9efa`
- Title: "Why Depth Matters in Parallelizable Sequence Models: A Lie Algebraic View"

## Assigned Role

Independent Reproducer A: careful experimentalist starting from the paper text, official artifacts, and the linked repository.

## Task Scope

I attempted the smallest feasible reproduction unit for the paper's empirical state-tracking claim: the documented word-problem pipeline under `state_tracking/readme.md`. The central claim tested was that finite-group state-tracking data and training scripts support the reported depth-dependent behavior, especially for symbolic word problems such as `S3` and `A5`.

I did not attempt a full Table 1 or Figure 3/4 reproduction because the paper reports 500k training sequences, 100 epochs for word problems, three seeds, and A100/H100 single-GPU runs. The available environment has no detected NVIDIA runtime or CUDA compiler.

## Evidence Examined

- Paper source: `artifacts/main.tex`
  - Abstract claims depth corresponds to Lie-algebra extensions and that approximation error diminishes exponentially with depth.
  - Section 4 claims experiments on symbolic word problems and a continuous-valued `A5` rotation task validate the theory.
  - Table 1 reports sequence-level accuracy for length-128 training and length-256 testing with 500k training sequences except `C2`.
  - Figure 3 reports maximum `A5` sequence length with `>90%` training sequence accuracy by depth.
  - Figure 4 reports depth-dependent MSE for the `A5` rotation task.
- Appendix source: `artifacts/A3_Experiments.tex`
  - Reports 100 epochs for word problems, 50 epochs for `A5` rotation, three seeds, 500k training sequences, batch sizes up to 2048, and single A100/H100 runs.
- Repository: `repos/lie-algebra-state-tracking` at `e6575fae9d3aa8f32cf30254269054fbf58c87b1`.
- README instructions:
  - `state_tracking/readme.md`
  - root `README.md`
  - `state_tracking/sub_example_job.sh`
- Relevant code:
  - `state_tracking/src/generate_data.py`
  - `state_tracking/src/main.py`
  - `state_tracking/src/model.py`
- Artifact availability:
  - Official artifacts include paper TeX/PDF and PNG figures.
  - I found no raw result tables, logs, checkpoints, W&B exports, or sweep scripts for the reported numerical results.

## Commands, Code, Derivations, or Literature Checked

All commands were run from `/home/mila/l/lia/peer-review-agents/agent_configs/agent1` unless a more specific working directory is shown. I used only the paper, official artifacts, linked repository, and required Koala platform guide. No OpenReview, citation, decision, social, or post-publication signals were used.

### Repository and Environment

```bash
git -C papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking rev-parse HEAD
```

Observed:

```text
e6575fae9d3aa8f32cf30254269054fbf58c87b1
```

```bash
python --version
python -c "import torch, numpy, polars; print('torch', torch.__version__); print('cuda_available', torch.cuda.is_available())"
python -c "import accelerate, datasets, tokenizers, transformers, fire, pyrootutils, wandb; print('core imports ok')"
python -c "import abstract_algebra, sfirah; print('abstract_algebra and sfirah ok')"
python -c "import fla; print('fla ok')"
python -c "import mamba_ssm; print('mamba_ssm ok')"
```

Observed:

```text
Python 3.12.12
ModuleNotFoundError: No module named 'torch'
ModuleNotFoundError: No module named 'accelerate'
ModuleNotFoundError: No module named 'abstract_algebra'
ModuleNotFoundError: No module named 'fla'
ModuleNotFoundError: No module named 'mamba_ssm'
```

```bash
nvidia-smi
nvcc --version
gcc --version
```

Observed:

```text
nvidia-smi: command not found
nvcc: command not found
gcc (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0
```

This does not match the README note that the authors used `gcc/12.2.0` and `cuda/12.4.1`.

### Minimal Dependency Environment

The active project Python had no `pip`, so I used a throwaway `uv` environment to avoid modifying the repo environment:

```bash
uv venv /tmp/koala-irA-230fcebb
uv pip install --python /tmp/koala-irA-230fcebb/bin/python fire polars pyrootutils git+https://github.com/jopetty/abstract_algebra
```

This installed the data-generation dependencies successfully.

### Data Generation Sanity Check

The documented scripts require a `.project-root` marker for `pyrootutils.find_root`. I created it temporarily in `state_tracking/` and removed it after the checks.

First, I tried to keep generated data outside the repo:

```bash
/tmp/koala-irA-230fcebb/bin/python src/generate_data.py --group=S3 --k=4 --samples=8 --seed=123 --data_dir=/tmp/koala-irA-230fcebb-data --overwrite=True
```

Observed:

```text
TypeError: unsupported operand type(s) for /: 'str' and 'str'
```

Cause: `generate_data.py` annotates `data_dir` as `str | Path` but immediately computes `data_dir / f"{group}={k}.csv"`. Fire passes the CLI value as a string, so custom `--data_dir` is broken.

Using the default repo-local data directory, tiny S3 generation succeeded:

```bash
/tmp/koala-irA-230fcebb/bin/python src/generate_data.py --group=S3 --k=4 --samples=8 --seed=123 --overwrite=True
```

Observed:

```text
Using seed 123
allowed indices = range(0, 6)
num_elements = 6
Randomly sampling 8 sequences.
Writing data to `.../state_tracking/data/S3=4.csv`
```

First rows:

```text
seed,input,target
123,5 4 2 4,5 1 4 3
123,5 0 2 2,5 5 3 5
123,0 5 3 1,0 5 2 3
123,1 0 2 0,1 1 4 4
```

Tiny A5 generation also succeeded:

```bash
/tmp/koala-irA-230fcebb/bin/python src/generate_data.py --group=A5 --k=4 --samples=8 --seed=123 --overwrite=True
```

Observed:

```text
Using seed 123
allowed indices = range(0, 60)
num_elements = 60
Randomly sampling 8 sequences.
Writing data to `.../state_tracking/data/A5=4.csv`
```

First rows:

```text
seed,input,target
123,50 46 20 48,50 15 32 16
123,51 9 20 20,51 55 30 41
123,14 0 26 5,14 14 52 56
123,35 4 18 26,35 31 47 31
```

### Training Script Sanity Check

With the active environment, even a tiny documented-style training invocation failed immediately:

```bash
python src/main.py train --group=S3 --k=4 --k_test=4 --n_layers=1 --epochs=1 --batch_size=4 --seed=1 --lr=1e-3 --model_name=transformer --max_samples=8 --logging=False
```

Observed:

```text
ModuleNotFoundError: No module named 'fire'
```

I then installed the repository `requirements.txt` into the same throwaway `uv` environment:

```bash
uv pip install --python /tmp/koala-irA-230fcebb/bin/python -r requirements.txt
```

Observed installed key versions:

```text
torch 2.11.0+cu130
cuda_available False
datasets 4.8.4
transformers 5.6.2
tokenizers 0.22.2
accelerate 1.13.0
```

The external model packages required by the root README were still unavailable:

```bash
/tmp/koala-irA-230fcebb/bin/python -c "import fla; print('fla ok')"
/tmp/koala-irA-230fcebb/bin/python -c "import mamba_ssm; print('mamba_ssm ok')"
/tmp/koala-irA-230fcebb/bin/python -c "import wavesAI; print('wavesAI ok')"
```

Observed:

```text
ModuleNotFoundError: No module named 'fla'
ModuleNotFoundError: No module named 'mamba_ssm'
ModuleNotFoundError: No module named 'wavesAI'
```

Even before model construction, the tiny training run failed in dataset preprocessing:

```bash
/tmp/koala-irA-230fcebb/bin/python src/main.py train --group=S3 --k=4 --k_test=4 --n_layers=1 --epochs=1 --batch_size=4 --seed=1 --lr=1e-3 --model_name=transformer --max_samples=8 --train_size=0.75 --logging=False
```

Observed:

```text
RuntimeWarning: FLA Models not available
RuntimeWarning: Mamba not available
RuntimeWarning: AUSSM not available
Constructing dataset from:
  .../state_tracking/data/S3=4.csv
ValueError: Column name ['token_type_ids'] not in the dataset. Current columns in the dataset: ['input', 'target', 'input_ids', 'labels']
```

Cause: `main.py` unconditionally calls `.remove_columns(["input", "target", "token_type_ids"])`, but the installed tokenizer stack from unpinned `requirements.txt` did not produce `token_type_ids`.

After testing, I removed the temporary repo-local `.project-root`, `data/` CSVs, and `src/__pycache__`; the repo clone returned to a clean state.

## Findings

1. **Smallest data-generation unit: partial match.** The finite-group data generator can create tiny `S3` and `A5` sequence-labeling CSVs using the official code and documented group names. This partially supports the availability of the data-generation component.

2. **Custom data directory is broken.** The documented CLI exposes `--data_dir`, but passing it as a normal path string fails because the code treats it as a `Path`. This is a small but concrete reproducibility defect because it makes safe scratch/output isolation fail without code changes.

3. **The training script is not reproducible from the supplied requirements as written.** A fresh install from `requirements.txt` pulled unpinned current versions (`transformers 5.6.2`, `tokenizers 0.22.2`, `datasets 4.8.4`), and `src/main.py train` failed before model construction due a missing `token_type_ids` column.

4. **Reported model runs require additional undocumented-or-external setup beyond `requirements.txt`.** The root README says to install `flash-linear-attention`, AUSSM, and Mamba-related code separately. After `requirements.txt`, `fla`, `mamba_ssm`, and `wavesAI` were still unavailable. Therefore the documented `transformer`, `gla`, `deltaproduct`, `mamba_lstm`, `neg_mamba_lstm`, and `aussm` model paths could not be executed in this pass.

5. **The environment cannot support the claimed full-scale experiments.** The paper says experiments used one A100 or H100 and no run exceeded three hours. This environment has no `nvidia-smi` or `nvcc`, and CUDA was unavailable to PyTorch in the throwaway environment.

6. **Raw reported metrics are not independently checkable from artifacts.** The official artifacts include static PNG figures and TeX tables, but I found no raw result CSVs, checkpoints, logs, exact sweep manifests, or W&B exports that would allow validating Table 1, Figure 3, or Figure 4 without rerunning the full experiments.

## Role-by-Role Findings

- Independent Reproducer A outcome: **partial for data generation, blocked for training/metrics reproduction**.
- No Independent Reproducer B reasoning was read or used.

## Match, Partial Match, Mismatch, or Blocked

Overall status: **Blocked for the paper's central empirical claim; partial match for the data-generation subcomponent.**

The tiny S3/A5 data-generation command produced plausible CSVs with the expected group cardinalities (`S3`: 6 elements, `A5`: 60 elements). However, I could not reproduce even a one-epoch, eight-sample training sanity run from the documented commands and requirements. The blocking failure is not merely lack of GPU: the Python data pipeline fails under a fresh requirements install before model construction, and the model packages required by the README are not installed by `requirements.txt`.

## Limitations or Blockers

- I did not patch code, pin older dependency versions, or manually install non-requirements model libraries beyond the documented `requirements.txt`, because the purpose of this pass was to assess reproducibility from the supplied paper/repo instructions.
- I did not run the full 500k-sequence, 100-epoch, three-seed experiments because the environment lacks the specified GPU/CUDA stack and the training script does not pass a tiny sanity run.
- I did not independently verify the numerical values in Table 1, Figure 3, or Figure 4 because raw logs/results/checkpoints were absent and the executable training path was blocked.

## Confidence Level

High confidence that the data generator can produce tiny group word-problem files. High confidence that the documented training path is currently not reproducible from the provided requirements in this environment. Moderate confidence about the broader empirical claim: the artifacts are insufficient for me to validate or falsify the reported depth trends, but the failure to run a tiny sanity job materially weakens reproducibility.

## Decision Impact

This should materially reduce confidence in the empirical support for the paper. The theoretical story may still be interesting, but the public artifact does not allow an independent reviewer to reproduce even a tiny training run without additional dependency archaeology and likely code changes. Under agent1's reproducibility-first rubric, I would mark the empirical component as **weakly reproducible**: data generation partially works, but model training and reported metrics are blocked. This pushes the score impact downward unless another role can reproduce the core depth trends from a pinned environment or provide raw logs/checkpoints tying the reported figures to executable code.
