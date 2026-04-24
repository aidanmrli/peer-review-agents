# Independent Reproducer A Report

Paper: `$V_1$: Unifying Generation and Self-Verification for Parallel Reasoners`  
Koala paper ID: `0a07cb4f-a3fc-42bd-988a-470a16f100e8`  
Role: Independent Reproducer A  
Date: 2026-04-24  

## Claim Attempted

I attempted to reproduce the smallest feasible unit of the paper's central empirical claim from the paper source and official repository:

> Pairwise self-verification via `V_1`-Infer ranks multiple candidate solutions better than pointwise verification and improves Pass@1 on reasoning benchmarks.

The paper source states this claim in `sections/abs.tex`, including that `V_1`-Infer improves Pass@1 by up to 10% over pointwise verification on code and math benchmarks. The repository README gives expected N=16, 3N-budget results, including:

- GPT-OSS-20B on LiveCodeBench-v6: Pass@1 61.4%, pointwise 71.8%, pairwise 76.3%.
- GPT-OSS-20B on AIME 2025: Pass@1 71.9%, pointwise 83.3%, pairwise 86.7%.
- Qwen3-4B-Instruct on LiveCodeBench-v6: Pass@1 35.4%, pointwise 38.9%, pairwise 43.5%.
- Qwen3-4B-Instruct on AIME 2025: Pass@1 45.4%, pointwise 53.3%, pairwise 63.3%.

Because full reproduction requires large model inference, SGLang/Modal GPU execution, real parquet datasets, and installed dependencies unavailable in this environment, I focused on:

1. Running a documented command on the smallest possible sample until the first reproducibility blocker.
2. Verifying that the repository's local pairwise aggregation formula ranks a synthetic candidate set consistently with the described margin-weighted win-rate mechanism.

## Setup Used

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Official repository path:

```bash
papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/
```

Repository commit:

```bash
git -C papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification rev-parse HEAD
# be5595566a7f19ae785f1695d05cff1e0c7db834
```

Python:

```bash
python --version
# Python 3.12.12
```

Documented dependencies were not installed in the active environment:

```bash
python - <<'PY'
import importlib.util
mods=['numpy','polars','pandas','hydra','omegaconf','transformers','sglang','openai','tqdm','termcolor','tabulate','sympy','datasets','huggingface_hub']
for m in mods:
    print(f'{m}:', 'ok' if importlib.util.find_spec(m) else 'missing')
PY
```

Observed output:

```text
numpy: missing
polars: missing
pandas: missing
hydra: missing
omegaconf: missing
transformers: missing
sglang: missing
openai: missing
tqdm: missing
termcolor: missing
tabulate: missing
sympy: missing
datasets: missing
huggingface_hub: missing
```

The checked-in dataset files are Git LFS pointer files, not usable parquet files:

```bash
for f in data/*.parquet; do echo "$f"; sed -n '1,5p' "$f"; done
```

Observed output:

```text
data/aime_2025.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:19df862b0e0c8cb826c6637a32a4d3c914019af009ed033493e3003a08a2a35f
size 17846
data/hmmt_feb_2025.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:d4dd4ed78f0d97daf811cf7ab63827cbfc8d6105cc37eaed95fc2c5847503f85
size 12774
data/test_code_contests.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:de56a100ec98584b4988ff590f5370d36805c7038646d03b9b6e16d42b5c2f6b
size 17278656
data/test_livecodebench_v6.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:645f8b8e4271151cd43206e91919164c58b9a68b532a2ce86dd2cf4f7dac8e49
size 118037264
```

No parquet reader was available locally:

```bash
python - <<'PY'
import importlib.util
for m in ['pyarrow','duckdb','fastparquet']:
    print(f'{m}:', 'ok' if importlib.util.find_spec(m) else 'missing')
PY
```

Observed output:

```text
pyarrow: missing
duckdb: missing
fastparquet: missing
```

## Documented Command Attempt

I selected the smallest documented math command variant, reducing `TOTAL_SAMPLES` to `1` and `N_PASSES` to `2` to avoid unnecessary model work if the pipeline started:

```bash
cd papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification
./eval/scripts/run_e2e_pairwise_math.sh "Qwen/Qwen3-4B-Instruct-2507" "" "aime_2025" 0 1 3.0 false medium 1.0 false 1234 8 0.1 2 false
```

Observed output:

```text
Downloading aime_2025.parquet from HuggingFace (harman/v1-verification-datasets)...
Traceback (most recent call last):
  File "<string>", line 2, in <module>
ModuleNotFoundError: No module named 'huggingface_hub'
```

Outcome: blocked before any dataset loading, candidate generation, verification call, or metric computation.

Concrete reason: the repository stores datasets as Git LFS pointers and depends on `huggingface_hub` to fetch the real files when pointer files are detected. The active environment lacks `huggingface_hub` and the rest of the documented stack. Even after dependency installation, the full claim would still require running Qwen/GPT-OSS model inference through SGLang or Modal/GPU infrastructure.

## Smallest Local Reproduction Unit

I reproduced the margin-weighted pairwise aggregation mechanism on a synthetic tournament using the formulas in `eval/verify_utils.py`:

- `_sanitize_pairwise_outcome`: clamp ratings to 1..10 and derive the winner from relative ratings.
- `_update_pairwise_aggregates`: use `weight = max(abs(rA-rB)/9, tau)` with `tau=0.1`.
- `_mu`: compute candidate score as weighted wins divided by total comparison weight.
- final ranking: sort by descending `mu`, then descending average rating.

Command:

```bash
python - <<'PY'
def sanitize(outcome):
    rA = max(1, min(10, int(outcome.get('rating_A', 5))))
    rB = max(1, min(10, int(outcome.get('rating_B', 5))))
    if rA > rB:
        w = 'A'
    elif rB > rA:
        w = 'B'
    else:
        w = 'tie'
    return w, rA, rB

def update(i, j, outcome, wins, W, rating_sum, rating_cnt, tau=0.1):
    w_sane, rA, rB = sanitize(outcome)
    margin = abs(rA-rB) / 9.0
    weight = max(margin, tau)
    p_ij = 0.5 if w_sane == 'tie' else (1.0 if w_sane == 'A' else 0.0)
    wins[i] += weight * p_ij
    wins[j] += weight * (1.0 - p_ij)
    W[i] += weight
    W[j] += weight
    rating_sum[i] += rA; rating_cnt[i] += 1
    rating_sum[j] += rB; rating_cnt[j] += 1

def mu(i, wins, W):
    return wins[i]/W[i] if W[i] > 0 else 0.5

def avg(i, rating_sum, rating_cnt):
    return rating_sum[i]/rating_cnt[i] if rating_cnt[i] > 0 else 0.0

n = 3
wins = [0.0]*n; W=[0.0]*n; rating_sum=[0]*n; rating_cnt=[0]*n
pairs = [
    (0,1, {'rating_A': 4, 'rating_B': 8}),
    (1,2, {'rating_A': 7, 'rating_B': 3}),
    (0,2, {'rating_A': 5, 'rating_B': 5}),
]
for i,j,o in pairs:
    update(i,j,o,wins,W,rating_sum,rating_cnt)
order = sorted(range(n), key=lambda t: (-mu(t,wins,W), -avg(t,rating_sum,rating_cnt)))
print('wins=', [round(x, 6) for x in wins])
print('weights=', [round(x, 6) for x in W])
print('mu=', [round(mu(i,wins,W), 6) for i in range(n)])
print('avg_rating=', [round(avg(i,rating_sum,rating_cnt), 6) for i in range(n)])
print('ranking=', order)
print('budget_n16_3x=', max(16, int(3.0*16)))
PY
```

Observed output:

```text
wins= [0.05, 0.888889, 0.05]
weights= [0.544444, 0.888889, 0.544444]
mu= [0.091837, 1.0, 0.091837]
avg_rating= [4.5, 7.5, 4.0]
ranking= [1, 0, 2]
budget_n16_3x= 48
```

This confirms a narrow implementation property: when one synthetic candidate wins both pairwise comparisons, the repository's documented margin-weighted win-rate rule ranks it first, and the N=16, 3x budget rule yields 48 pairwise verification calls, matching the paper text's "48 verification calls" for N=16 at 3x budget.

## Observed Result

Match status: partial local match for the aggregation arithmetic only; blocked for the empirical claim.

What I could verify:

- The official repository is present at the requested commit.
- The README and scripts document concrete N=16, 3x evaluation commands and expected results.
- The local pairwise aggregation rule behaves coherently on a synthetic tournament.
- The 3x budget rule maps N=16 to 48 pairwise verification calls, consistent with the paper's LiveCodeBench-v6 claim.

What I could not verify:

- I did not reproduce any reported benchmark accuracy.
- I did not run candidate generation, pairwise self-verification with an LLM judge, pointwise baseline verification, code grading, or math grading.
- I did not load the official datasets because the checked-in files are Git LFS pointers and the environment lacks the download/parquet dependencies.
- I did not use API keys, Modal, GPUs, or large model execution.

## Reproduction Outcome

Weak reproducibility for this pass. The smallest implementation-level unit of the pairwise ranking arithmetic is reproducible from the code, but the central empirical claim is not reproducible in the available environment from the checked-out artifacts alone. The immediate blockers are missing Python dependencies and unresolved Git LFS dataset pointers; the larger blocker is that full reproduction requires substantial model-serving infrastructure.

## Score Impact

This should materially limit confidence in the paper's empirical claims unless another internal role can run the full artifact stack or the authors provide a pinned, dependency-complete environment with real dataset files/checksums and cached generations/verifier outputs. The method's aggregation arithmetic is inspectable and locally sane, but the benchmark improvements over pointwise verification remain unverified by this independent reproduction attempt.
