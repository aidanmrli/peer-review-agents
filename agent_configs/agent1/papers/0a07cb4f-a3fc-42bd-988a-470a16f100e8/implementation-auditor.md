# Implementation Auditor Report

Paper: `$V_1$: Unifying Generation and Self-Verification for Parallel Reasoners`

Paper ID: `0a07cb4f-a3fc-42bd-988a-470a16f100e8`

Artifact audited:

- Paper source: `artifacts/source/`
- Official repository: `artifacts/repo/pairwise-self-verification/`
- Repository commit: `be5595566a7f19ae785f1695d05cff1e0c7db834`

## Claim Being Tested

The artifact should support the paper's central implementation claims:

1. `V_1`-Infer ranks parallel candidate solutions through pairwise self-verification using a Swiss-style tournament and improves Pass@1 over pointwise verification on code and math benchmarks.
2. `V_1`-PairRL jointly trains one model for generation and pairwise self-verification using GRPO/DAPO-style RL, sparse verifier rewards, a correct-containing pairing strategy, DeepCoder training data, and trained-model evaluations over multiple seeds.
3. The released scripts, configs, data, dependencies, and metrics should be sufficient to reproduce or audit the reported evaluation numbers.

## Artifact Inventory

The repository is primarily a `V_1`-Infer evaluation artifact, not a full training artifact.

Present:

- `README.md`: installation, quick-start commands, expected results, file structure.
- `config/generation.yaml`: Hydra defaults for generation/evaluation.
- `eval/eval_e2e_pairwise.py`: end-to-end candidate generation, pairwise verification, grading, metrics output.
- `eval/eval_e2e_pointwise.py`: analogous pointwise baseline path.
- `eval/verify_pairwise.py`: pairwise verifier, Swiss/min-degree/random ranking paths.
- `eval/verify_pointwise.py`: pointwise verifier.
- `eval/verify_utils.py`: verifier prompts, score parsing, aggregation, metric calculation.
- `eval/eval_results_parallel.py`: post-generation correctness grading and Pass@N/Pass@1 computation.
- `eval/scripts/*.sh`: code/math pairwise and pointwise launch scripts.
- `rewards/`: math and code grading utilities.
- `modal_eval.py`: Modal/H100 runner with dependency installation, data fallback download, and result upload.
- `scripts/run_e2e_commands_n16.sh`, `scripts/modal_eval_commands_n16.sh`: canned commands for N=16 inference evaluations.
- `assets/`: animations only.
- `data/*.parquet`: four Git LFS pointer files in this checkout, not real parquet payloads.

Absent:

- No `requirements.txt`, `pyproject.toml`, `environment.yml`, lockfile, Dockerfile, or pinned local environment file. Modal pins some packages, but the README does not.
- No RL training code for `V_1`-PairRL, PointRL, RL baseline, DAPO/GRPO trainer, rollout collector, verifier reward integration, checkpoint selection, or DeepCoder training/validation pipeline.
- No released trained checkpoints, checkpoint metadata, validation logs, run manifests, seeds for trained-model experiments, or raw result tables underlying the paper figures.
- No precomputed generations/caches/results for the paper's reported metrics.
- No `test_livecodebench.parquet` file is present in this checkout, despite README listing it as included.

## Code Paths Inspected

Commands run from the repository root unless noted:

```bash
git rev-parse HEAD
git status --short
find artifacts/source -maxdepth 3 -type f
find artifacts/repo/pairwise-self-verification -maxdepth 3 -type f
rg -n "train|GRPO|DAPO|PairRL|PointRL|DeepCoder|checkpoint|verl|deepspeed|accelerate|optimizer|learning_rate|lambda|sparsity|correct|incorrect|pair" .
sed -n '1,240p' README.md
sed -n '1,260p' eval/eval_e2e_pairwise.py
sed -n '1,620p' eval/verify_pairwise.py
sed -n '1,420p' eval/verify_utils.py
sed -n '1,340p' eval/eval_results_parallel.py
sed -n '1,260p' eval/utils.py
sed -n '1,220p' eval/scripts/run_e2e_pairwise.sh
sed -n '1,220p' eval/scripts/run_e2e_pairwise_math.sh
sed -n '1,320p' modal_eval.py
ls -lh data
file data/*.parquet
for f in data/*.parquet; do sed -n '1,20p' "$f"; done
bash -n eval/scripts/run_e2e_pairwise.sh eval/scripts/run_e2e_pairwise_math.sh eval/scripts/run_e2e_pointwise.sh eval/scripts/run_e2e_pointwise_math.sh scripts/run_e2e_commands_n16.sh scripts/modal_eval_commands_n16.sh
python3 -m py_compile eval/verify_utils.py eval/verify_pairwise.py eval/eval_e2e_pairwise.py eval/eval_e2e_pointwise.py eval/eval_results_parallel.py eval/utils.py rewards/*.py
python3 - <<'PY'
mods=['numpy','pandas','hydra','omegaconf','transformers','sglang','openai','tqdm','termcolor','tabulate','sympy','datasets','huggingface_hub','modal','torch','polars','pyarrow']
for m in mods:
    try:
        __import__(m)
        print(m, 'OK')
    except Exception as e:
        print(m, 'MISSING/FAIL', type(e).__name__, str(e)[:80])
PY
```

Lightweight outcomes:

- Repository HEAD matched the requested commit.
- Shell scripts passed `bash -n`.
- Python files passed `py_compile`; this checks syntax but not imports.
- The local environment lacks essentially all runtime dependencies (`numpy`, `torch`, `polars`, `hydra`, `transformers`, `sglang`, `datasets`, `huggingface_hub`, etc.), so no end-to-end execution was attempted.
- The data files in the checkout are ASCII Git LFS pointer files, not readable parquet files. Example: `data/test_livecodebench_v6.parquet` is a 134-byte pointer with `size 118037264`.

## Paper-to-Code Matches

The released inference code does implement the broad `V_1`-Infer evaluation pattern described in the paper:

- The README describes `V_1`-Infer as pairwise comparison over candidate solutions with a Swiss tournament, margin-weighted win rate `mu`, and top-ranked selection.
- `eval/verify_utils.py` defines pairwise and pointwise code/math prompts, parses tagged ratings, derives the winner from `rating_A`/`rating_B`, and computes weighted win aggregates using a margin floor `tau`.
- `eval/verify_pairwise.py` implements min-degree coverage, Swiss-style pairing over nearby `mu` values, `random`, and `swiss_parallel` variants. The default launch scripts use `coverage_strategy=min_degree`, `min_degree=2`, `max_window=8`, and `tau=0.1`, matching the paper's appendix hyperparameters.
- `eval/eval_e2e_pairwise.py` generates N samples, calls the pairwise verifier, grades all generated solutions against reward data, computes Pass@N/Pass@1, computes selected-solution verification scores, and writes `metrics.json`.
- Code launch scripts set temperature `0.6`, top-p `0.95`, max response length `32768`, SGLang inference, N candidates, budget multiplier, seed, and code prompt template, consistent with the paper's stated inference settings.
- Math scripts set temperature `1.0`, top-p `0.95`, SGLang inference, N candidates, and the same pairwise tournament parameters.
- `eval_results_parallel.py` computes generated-solution Pass@N as any correct solution among samples and Pass@1 as mean correctness over generated samples; selected-solution correctness is computed later in `eval_e2e_pairwise.py`/`eval_e2e_pointwise.py`.

## Paper-to-Code Discrepancies

### 1. The artifact does not contain the PairRL implementation.

This is the major discrepancy. The paper's Section 5 claims a unified RL training method: GRPO/DAPO-style training, `J_gen + lambda J_pairverif`, sparse verifier rewards, correct-containing pair construction, DeepCoder training, baseline RL, PointRL, non-co-evolving ablation, 150 steps, checkpoint selection by validation Pass@1, and trained-model results over three seeds. The repository contains reward utilities and inference/evaluation scripts, but no trainer, no GRPO/DAPO/verl/rLLM configuration, no DeepCoder training scripts, no verification reward implementation wired into training, no PointRL training path, no baseline RL training path, no non-co-evolving training path, no checkpoint selection code, and no checkpoints.

The README explicitly scopes the repo as "Code for `V_1`-Infer" and does not document `V_1`-PairRL training. Therefore the strongest paper claims about unified generation/self-verification training are not independently auditable from the artifact.

### 2. The advertised datasets are not actually included in this checkout.

The README says "All evaluation datasets are included in `data/`", but the checkout contains Git LFS pointers only:

- `aime_2025.parquet`: 130-byte ASCII pointer, target size 17,846 bytes.
- `hmmt_feb_2025.parquet`: 130-byte ASCII pointer, target size 12,774 bytes.
- `test_code_contests.parquet`: 133-byte ASCII pointer, target size 17,278,656 bytes.
- `test_livecodebench_v6.parquet`: 134-byte ASCII pointer, target size 118,037,264 bytes.

The scripts can download from Hugging Face if a file is missing or smaller than 10 KB. That may recover the files for users with network access and correct dependencies, but the artifact as checked out is not self-contained. The absent `test_livecodebench.parquet` file for LiveCodeBench-v5 is also inconsistent with the README file-structure listing.

### 3. Dependency documentation is incomplete and partly unpinned.

The README gives a one-line unpinned `pip install` command. It omits packages used by the code paths, including at least `datasets`, `huggingface_hub`, `torch`, `litellm`, `scipy`, `antlr4-python3-runtime`, `pylatexenc`, `regex`, and `math-verify`. `modal_eval.py` has a more complete environment and pins `torch==2.5.1` and `sglang[all]==0.5.1.post1`, but most other packages remain unpinned. There is no lockfile or reproducible environment spec for local reproduction.

### 4. Reported metrics are not backed by raw outputs.

The README lists expected results for a subset of N=16, budget 3N experiments, and the paper contains many more figures and comparisons. The repo has scripts that would compute metrics after expensive generation/verification, but it does not include raw generations, judge outputs, scores, `metrics.json`, result parquet shards, caches, logs, or figure-generation data. Consequently, the artifact supports re-running some inference experiments in principle, but not independently checking that the reported figures were computed from the claimed runs.

### 5. Inference code depends on substantial hidden service assumptions.

The default path starts an SGLang OpenAI-compatible server locally, selects ports dynamically, sets `api_key="EMPTY"`, and drives high-concurrency threaded calls. `modal_eval.py` assumes Modal, H100 GPUs, Modal secrets named `hf-token` and `github-token`, persistent volumes, and optional `HF_RESULTS_REPO`. These assumptions are workable for the authors' environment but underdocumented for independent reproduction.

### 6. The local scripts do not cover all paper experiments.

The command scripts cover selected N=16 inference commands for GPT-OSS-20B and Qwen3-4B-Instruct over math/code benchmarks. The paper also discusses GPT-OSS-120B, Qwen3-4B-Thinking, SWE-Bench, RSA combinations, N=8, budget-matched N=32 pointwise runs, budget curves, qualitative examples, and all trained-model/ablation results. Some scripts may be adaptable, but the artifact does not provide a complete executable manifest for every figure/table.

## Reproducibility Blockers

High-impact blockers:

- Missing PairRL/PointRL/baseline RL training implementation and checkpoints. This blocks independent verification of the paper's central "unifying generation and self-verification" training claim.
- Missing raw experiment outputs and metrics. This blocks audit of reported numbers without rerunning large jobs.
- Data files are LFS pointers in this checkout; reproducing requires Git LFS or Hugging Face downloads.
- No local reproducible environment file; README dependencies are incomplete and unpinned.
- Local environment had none of the runtime dependencies installed, so only static/syntax checks were possible here.
- Reproduction requires large model inference with SGLang and H100-class GPUs. README expected results state "3x H100 GPUs via Modal"; this is credible for authors but not lightweight.

Medium-impact blockers:

- The scripts use very high concurrency (`data.num_workers=256`, thread pools over samples and pairwise calls), which may be fragile across servers and GPU memory settings.
- Cache keys include `config.data.output_path` in verifier prompts' cache key construction, making cache reuse sensitive to output directory names.
- Data auto-download depends on `huggingface_hub`, which is omitted from README installation.
- The Modal runner clones the live GitHub repo at depth 1 rather than explicitly checking out the audited commit, so future use of `modal_eval.py` may not reproduce commit `be5595566a7f19ae785f1695d05cff1e0c7db834` unless the repository state is controlled externally.

## Severity for Acceptance Decision

Severity: high.

The inference artifact partially supports the `V_1`-Infer mechanism and contains plausible scripts for rerunning selected base-model pairwise vs pointwise evaluations. However, the artifact does not support the paper's strongest and most novel training-side claims: `V_1`-PairRL, co-evolving unified generation/verifier training, sparse verifier rewards in RL, trained-model gains, PointRL/RL baselines, and co-evolving ablations. Those claims materially affect the paper's acceptance case and cannot be independently verified from the released implementation.

My implementation-audit score impact is a substantial downgrade on reproducibility. The artifact is acceptable as a partial inference demo/evaluation harness, but it is not a complete implementation release for the paper as written.

## Concise Final Synthesis

The repository at the requested commit is best characterized as a `V_1`-Infer evaluation harness. It implements pairwise and pointwise self-verification pipelines, Swiss/min-degree ranking, benchmark grading, and Modal launch scripts. It does not contain the RL training system, trained checkpoints, raw results, complete data payloads, or reproducible environment needed to audit the paper's unified PairRL claims. The implementation artifacts therefore support only a subset of the inference claims and leave the core training-result claims unreproduced.
