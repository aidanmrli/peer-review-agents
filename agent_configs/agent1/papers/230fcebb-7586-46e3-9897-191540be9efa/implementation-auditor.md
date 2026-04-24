# Implementation Auditor Report

Paper ID: `230fcebb-7586-46e3-9897-191540be9efa`

Title: "Why Depth Matters in Parallelizable Sequence Models: A Lie Algebraic View"

Assigned role: Implementation Auditor

Repository inspected: `papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking`

Repository HEAD: `e6575fae9d3aa8f32cf30254269054fbf58c87b1`

## Task Scope

I inspected whether the released repository and artifact bundle support independent reproduction of the paper's experimental claims: Table 1 word-problem generalization, Figure 2 A5 depth-vs-length behavior, and Figure 3/A4 rotated-vector MSE curves. I focused on artifact completeness, installability, dependency pinning, data generation, model-code alignment, run configuration, logging, and whether the released code contains enough raw outputs to recreate the reported figures.

I did not run the full experiments. The current environment lacks the required Python/GPU stack (`torch`, `accelerate`, `datasets`, `fla`, `mamba_ssm`, `wavesAI`, `wandb`, etc.), and the paper's own setup requires CUDA compilation plus external packages. I performed static code inspection and lightweight syntax/import checks without modifying the repository.

## Evidence Examined

- Paper source and PDF artifacts:
  - `artifacts/main.tex`
  - `artifacts/A3_Experiments.tex`
  - `artifacts/figures/seqlen_vs_depth_vertical.png`
  - `artifacts/figures/mse_grid.png`
  - `artifacts/figures/mse_grid_aussm.png`
  - `artifacts/paper.pdf`
- Repository files:
  - `README.md`
  - `state_tracking/readme.md`
  - `state_tracking/requirements.txt`
  - `state_tracking/sub_example_job.sh`
  - `state_tracking/src/generate_data.py`
  - `state_tracking/src/main.py`
  - `state_tracking/src/main_regression.py`
  - `state_tracking/src/model.py`
  - `state_tracking/src/utils.py`
  - `mamba_dev/setup.py`
  - `mamba_dev/pyproject.toml`
  - `mamba_dev/mamba_ssm/ops/selective_scan_interface.py`
  - `diff_AUSSM.txt`

Commands run:

```bash
git rev-parse HEAD
find papers/230fcebb-7586-46e3-9897-191540be9efa/artifacts -maxdepth 3 -type f -printf '%p %s bytes\n'
rg -n "GitHub|code|reproduc|seed|Table|Figure|experiment|AUSSM|Mamba|depth|MSE|compute" artifacts/main.tex artifacts/A3_Experiments.tex
rg --files papers/230fcebb-7586-46e3-9897-191540be9efa
rg -n "wandb|plot|figure|mse_grid|seqlen_vs_depth|best_sequence|sweep|seed|1000|500000" .
find . -maxdepth 4 \( -iname '*result*' -o -iname '*figure*' -o -iname '*plot*' -o -iname '*sweep*' -o -iname '*csv' -o -iname '*jsonl' -o -iname '*wandb*' -o -iname '*log' \) -print
PYTHONDONTWRITEBYTECODE=1 python - <<'PY'
import ast, pathlib
for p in [
  pathlib.Path('state_tracking/src/generate_data.py'),
  pathlib.Path('state_tracking/src/main.py'),
  pathlib.Path('state_tracking/src/main_regression.py'),
  pathlib.Path('state_tracking/src/model.py'),
  pathlib.Path('state_tracking/src/utils.py'),
]:
    ast.parse(p.read_text())
    print(f'AST OK {p}')
PY
PYTHONDONTWRITEBYTECODE=1 python - <<'PY'
import importlib.util
mods=['torch','accelerate','datasets','fire','polars','pyrootutils','sfirah','abstract_algebra','fla','mamba_ssm','wavesAI','wandb','tokenizers','transformers']
for m in mods:
    print(f'{m}: {"present" if importlib.util.find_spec(m) else "missing"}')
PY
git ls-remote https://github.com/jopetty/abstract_algebra.git HEAD
git ls-remote https://github.com/jopetty/sfirah.git HEAD
git ls-remote https://github.com/arjunkaruvally/AUSSM.git HEAD
```

## Artifact Inventory

The artifact bundle contains the paper source, bibliography, PDF, and final plotted image files. It does not contain experiment raw data, generated train/test CSVs, W&B run exports, checkpoint files, sweep manifests, exact run commands per reported number, or plotting/aggregation scripts that transform raw logs into the submitted figures.

The repository contains:

- A top-level `README.md` with external setup notes.
- A `state_tracking` experiment directory with data generation, classification training, regression training, one Slurm example, and loose requirements.
- A nested `mamba_dev` fork with modified Mamba code and custom CUDA extensions for positive-only and positive-and-negative selective scans.
- A patch file `diff_AUSSM.txt` for the external AUSSM repository.

No `data/`, `logs/`, `wandb/`, `checkpoints/`, `run_checkpoints/`, result CSV, JSONL, or figure-generation scripts are present in the repo clone.

## Code Paths Inspected

- Word-problem data generation: `state_tracking/src/generate_data.py`
  - Generates finite group sequence-labeling CSVs from `abstract_algebra`.
  - Supports `Z`, `D`, `H`, `S`, and `A` group identifiers.
  - Random sampling is controlled by `--seed`, but README examples omit the data seed and the default seed is random.
- Word-problem training/evaluation: `state_tracking/src/main.py`
  - Loads CSV files, tokenizes them, prepends BOS, trains with cross-entropy, computes token and sequence accuracy, and logs W&B tables.
  - Implements `deltaproduct`, `gla`, `transformer`, `mamba_lstm`, `neg_mamba_lstm`, and `aussm`.
- Rotated-vector regression: `state_tracking/src/main_regression.py`
  - Reuses the A5 CSV sequence data but replaces labels with GPU-built 3D rotation targets.
  - Uses the paper's P and R generator matrices and an embedded/full `A5.json` Cayley table.
  - Replaces LM heads with 3D regression heads and logs per-position MSE curves.
- Model wrappers: `state_tracking/src/model.py`
  - Wraps FLA models, local Mamba, and external AUSSM `wavesAI.model.aussm.SSMSeq2Seq`.
- Custom signed Mamba path: `mamba_dev/mamba_ssm/ops/selective_scan_interface.py`
  - Dispatches to `selective_scan_cuda_positive` or `selective_scan_cuda_positive_and_negative`.
- Mamba build: `mamba_dev/setup.py`
  - Defines the two custom CUDA extensions but also includes an upstream-wheel cache path.

## Paper-Code Matches

- The paper states that word problems are causal sequence-labeling tasks with a BOS token. The code prepends a BOS token with `TemplateProcessing` and trains token-level labels at every position.
- The paper states the training/test lengths are 128/256 for word problems. The training commands expose `--k` and `--k_test`, and the example uses `trainLen=128`, `evalLen=256`.
- The paper states the hyperparameter space includes hidden size, learning rate, and batch size. The training entry points expose `--hidden_size`, `--lr`, and `--batch_size`.
- The paper states transformer/GLA/DeltaProduct use `flash-linear-attention`; `state_tracking/src/main.py` imports FLA `TransformerConfig`, `GLAConfig`, and `GatedDeltaProductConfig`.
- The paper distinguishes positive-only Mamba and signed Mamba. The code uses `mamba_lstm` with `positive_and_negative_associative_scan=False` and `neg_mamba_lstm` with `True`.
- The paper's A5 rotation matrices match the matrices hard-coded in `main_regression.py`.
- The code has logging fields for sequence-level accuracy and maximum prefix length at 90 percent, which correspond to the quantities described for Figure 2.
- AST parsing succeeded for the key Python files, so the inspected Python source is syntactically valid.

## Paper-Code Discrepancies and Reproducibility Blockers

### High severity: no raw results or figure/table reconstruction path

The paper reports Table 1 and multiple figures with best-of-sweep values and standard errors over three seeds. The repository contains no raw run outputs, no W&B export files, no result CSVs, no aggregation scripts, and no plot scripts. The only published plot artifacts are final image files in the paper bundle. The training code logs to W&B, but reviewers are not given the W&B project, exported tables, run IDs, or a script to reconstruct `seqlen_vs_depth_vertical.png`, `mse_grid.png`, or `mse_grid_aussm.png`.

Decision impact: This is the largest implementation-audit failure. The code may be capable of generating similar logs, but the reported numerical claims cannot be independently traced from released artifacts to paper tables/figures.

### High severity: hyperparameter sweeps are claimed but not scripted

Appendix A3 says Table 1 values are the best results from all combinations in hidden size `{64,128,256}`, learning rate `{1e-2,1e-3,3e-4,1e-4}`, batch size `{256,2048}`, three random seeds, and 100 epochs. The repo provides only one example Slurm script for `task=S3`, `model=gla`, `numLayers=1`, `batch=2048`, `lr=1e-3`, `seed=1`. The README suggests looping only over seeds, not over the full grid, models, groups, depths, train/test lengths, or Householder settings.

Decision impact: Table 1 cannot be reproduced without reverse-engineering a large run matrix. This materially weakens confidence in every "best result" cell, especially failures reported as exactly `0.00` or partial `0.19`.

### High severity: dependency stack is not pinned and not fully specified

`state_tracking/requirements.txt` uses lower bounds, not exact versions. It also omits major required packages for the reported experiments, including FLA, local `mamba_dev`, `causal-conv1d`, and AUSSM/wavesAI. The README names FLA commit `8d25cac1e1779dad0f1e36cb033c5a9b10bf9c01`, but this is not enforced in a requirements file, lockfile, install script, or submodule. AUSSM is only specified by an external repository plus a patch file, with no pinned AUSSM commit in the repository instructions. The two Git dependencies in requirements, `jopetty/abstract_algebra` and `jopetty/sfirah`, are unpinned and will track whatever branch HEAD is available at install time.

The current audit environment had none of the required Python packages installed, confirming that a clean reviewer environment cannot run the scripts without substantial manual setup.

Decision impact: The released implementation is not environment-reproducible. Version drift in FLA, `abstract_algebra`, `sfirah`, Mamba, AUSSM, CUDA, and Torch can change both correctness and trainability.

### High severity: Mamba install can silently use upstream wheel instead of local modified kernels

The signed-Mamba result depends on custom local CUDA extensions named `selective_scan_cuda_positive` and `selective_scan_cuda_positive_and_negative`. However, `mamba_dev/setup.py` still points `BASE_WHEEL_URL` to upstream `https://github.com/state-spaces/mamba/releases/...` and its `CachedWheelsCommand` downloads a prebuilt upstream wheel unless `MAMBA_FORCE_BUILD=TRUE`. If a compatible upstream wheel exists, installation can bypass the local modified source, likely removing the custom signed/positive scan behavior needed for the paper's Mamba variants.

Decision impact: This is a concrete artifact hazard for the paper's signed-Mamba claims. The README does not instruct reviewers to force a source build, so a nominal installation may not implement the architecture used in the paper.

### Medium to high severity: training/evaluation sample counts do not directly match the paper

The paper states 500K training sequences for all non-C2 tasks and 1000 test sequences. The code default `train_size=0.99` means a generated 500000-row training CSV yields 495000 training rows and 5000 held-out rows. For `k_test=256`, the same split logic is applied to the test-length CSV and evaluation uses `dataset_test["test"]`, i.e. 1 percent of whatever file was generated. To obtain exactly 1000 evaluation examples, a reviewer would need to generate 100000 test-length samples, but the README does not say this. The README also does not specify the 100K C2 exception from the paper.

Decision impact: Even with the right code, a reviewer following the README will not necessarily reproduce the paper's dataset sizes or evaluation protocol.

### Medium to high severity: data-generation seeds are not disclosed

The paper says training/test datasets are randomly sampled. The data-generation script supports `--seed`, prints it, and writes a `seed` column, but the README examples omit the data seed and the repository does not provide generated CSVs or seed lists for the reported runs.

Decision impact: Exact reproduction of sampled train/test sets is impossible from the release. This matters for unstable/deep settings where the paper itself reports trainability issues.

### Medium severity: AUSSM sequence handling is under-specified and can drop the last token

The paper says the AUSSM backward kernel assumes sequence length is a multiple of 8. The README says "multiple of 8 (or 16? didn't check the detail)." The code in `main.py` and `main_regression.py` actually hard-codes a multiple-of-32 check after BOS insertion and, when the tensor length is not divisible by 32, asserts `(slen - 1) % 32 == 0` and truncates `source`/`target` by one token. For a nominal `k=128` word problem, BOS makes `slen=129`, so AUSSM is trained/evaluated on a truncated sequence of length 128 including BOS, dropping the final problem token. For `k_test=256`, it similarly drops the final token after BOS.

Decision impact: AUSSM rows are not strictly evaluated on the same token horizon as the other models unless the authors account for the BOS/truncation convention. The released documentation does not make this clear.

### Medium severity: Figure 2 prefix-length metric appears to include BOS

`cumulative_sequence_accuracies` computes prefix all-correct accuracy over the full target tensor, including BOS, and `max_prefix_at_threshold` returns `i + 1`. The paper describes maximum sequence length of the word problem. Unless postprocessing subtracts BOS, the logged "max sequence length" can be shifted by one relative to the problem-token length. No plotting script is present to determine whether the submitted figure corrected this.

Decision impact: This is a smaller numerical issue than the missing raw outputs, but it further blocks auditability of Figure 2.

### Medium severity: Mamba architecture includes convolution not described in the experimental setup

For both `mamba_lstm` and `neg_mamba_lstm`, the code sets `d_conv=4` and comments that the author is "leaving conv for Mamba for now as there is no option to remove it without touching the code." The paper frames the experiments as representative diagonal structured SSMs and emphasizes generator structure, but the released Mamba implementation includes a local convolutional component that may affect learnability and state-tracking behavior.

Decision impact: The practical Mamba experiments are not a clean test of only diagonal signed/positive SSM dynamics. This does not invalidate the experiments by itself, but it should be disclosed when interpreting algebraic claims from code.

### Medium severity: compute claim cannot be audited

Appendix A3 says all experiments used a single A100 or H100 and no run lasted longer than three hours. The example Slurm script requests one GPU and a 2-hour wall clock, but the repository has no timing logs, W&B run durations, hardware metadata, or per-run batch/throughput summaries. The full sweep implied by the paper is large: hidden-size/lr/batch/seed combinations across multiple groups, architectures, depths, plus A5 and rotation runs.

Decision impact: Per-run runtime may be plausible for small models on A100/H100, but the claim is not independently verifiable from the artifact release.

### Low to medium severity: documentation has ambiguous task naming

The paper reports `C3`, but the README example list says `Z60 (C60)` rather than documenting `Z3` for `C3`. The generator likely supports `Z3`, so this is probably a documentation error, but it adds friction for exact reproduction of Table 1.

## External Package and Commit Availability

I confirmed that `jopetty/abstract_algebra`, `jopetty/sfirah`, and `arjunkaruvally/AUSSM` have reachable HEADs via `git ls-remote`. This does not solve reproducibility because the release does not pin their commits. The FLA commit is documented in prose but not encoded in installation metadata. The Mamba fork is vendored locally, but its build script can select an upstream wheel unless forced to build from source.

## Severity Summary

Overall severity for acceptance decision: High.

The repository is a useful partial code release, not a reproducible experimental artifact. It contains plausible implementations of the task generator, training loops, A5 regression target construction, and model wrappers, but it lacks the operational material needed to verify the reported experimental results: exact data seeds, generated datasets, complete sweep scripts, locked dependencies, raw logs, aggregation code, and plot reconstruction. The most serious implementation risks are the missing run-to-figure provenance and the Mamba build path that may silently install upstream code instead of the modified signed-Mamba kernels.

## Decision Impact

The implementation audit materially reduces confidence in the empirical support for the paper's claims. The theoretical contribution may stand independently, but the experiments are decision-relevant because the paper uses them to argue that increasing depth systematically mitigates state-tracking error in practical parallelizable models. From the released artifacts, I cannot independently verify the exact Table 1 values, the Figure 2 depth-vs-length trend, or the Figure 3/AUSSM MSE curves. I would mark the empirical reproducibility as weak unless another role successfully reproduces the central curves from a fully reconstructed environment.

Confidence level: High for artifact-completeness findings; medium for code-behavior concerns that depend on runtime installation details; low for actual numerical reproducibility because the required GPU/dependency stack was not run.
