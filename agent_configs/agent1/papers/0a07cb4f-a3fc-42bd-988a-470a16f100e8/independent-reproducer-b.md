# Independent Reproducer B Report

Paper: `$V_1$: Unifying Generation and Self-Verification for Parallel Reasoners`

Paper ID: `0a07cb4f-a3fc-42bd-988a-470a16f100e8`

Role: Independent Reproducer B

Date: 2026-04-24

I did not read `independent-reproducer-a.md`. This report uses the paper source under `artifacts/source/` and the official repository under `artifacts/repo/pairwise-self-verification/` at commit `be5595566a7f19ae785f1695d05cff1e0c7db834`.

## Claim Attempted

I attempted to validate the central empirical/mechanistic claim that pairwise self-verification (`V_1` / `V_1-Infer`) gives a better selection signal than pointwise self-verification for parallel reasoning candidates, producing higher final-answer accuracy under controlled verification budgets.

The paper states this directly in `sections/method.tex`:

- Lines 151-154: the goal is to compare pairwise verification with pointwise verification, and the paper reports pairwise improvements on CodeContests, LiveCodeBench-v5, and HMMT.
- Lines 157-158: the paper claims improved test-time scaling versus RSA on LiveCodeBench-v6 with `N=16`.
- Lines 180-182: the paper claims larger gains on difficult problems and a 76.3% vs 72.5% ablation over random pairwise verification.

I treated the broader PairRL training claim as only partially checkable from this artifact, because the official repository appears to expose inference/evaluation code but not training code, trained checkpoints, or cached generated outputs.

## Independent Route Used

I used a code-level trace plus a dependency-free synthetic sanity check:

1. Verified the artifact commit and repository cleanliness.
2. Inspected paper claims in the LaTeX source.
3. Inspected the official implementation of pointwise scoring, pairwise weighted aggregation, and Swiss tournament pair selection.
4. Inspected the dataset artifacts and environment state.
5. Reimplemented the repo's aggregation and default Swiss pairing logic in a small standalone Python sanity check, avoiding model calls and third-party dependencies.

This is a different route from a full benchmark rerun: it validates whether the released algorithmic core can, in principle, realize the paper's claimed pointwise-vs-pairwise advantage when pairwise comparisons contain information that pointwise scores lose.

## Commands and Observations

### Repository commit and status

Command:

```bash
git -C papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification rev-parse HEAD
git -C papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification status --short
```

Observed:

```text
be5595566a7f19ae785f1695d05cff1e0c7db834
```

The checkout was at the requested commit. `status --short` emitted no changed files.

### Paper claim locations

Command:

```bash
nl -ba papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source/sections/method.tex | sed -n '148,184p'
```

Observed claim text includes:

- Pairwise verification is compared against pointwise verification, with reported gains on CodeContests, LiveCodeBench-v5, and HMMT.
- Pairwise verification is claimed to improve test-time scaling and reach 76% Pass@1 on LiveCodeBench-v6 with 48 verification calls.
- The random-pairing ablation is reported as 76.3% for `V_1-Infer` versus 72.5% for random pairing on LCB-v6 with GPT-OSS-20B at budget 3x.

### Official repo scope

Command:

```bash
find papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification -maxdepth 4 -type f | sort | sed -n '1,260p'
```

Observed:

- The repo contains `eval/`, `rewards/`, `config/`, `data/`, `modal_eval.py`, and shell scripts.
- I did not find PairRL training code, trained model checkpoints, cached generations, benchmark result parquets, or raw result tables sufficient to recompute the paper's numeric figures locally.
- The README describes the repo as code for `V_1-Infer`, not as a complete training reproduction package.

### Dependency and data blockers

Command:

```bash
python - <<'PY'
mods=['numpy','pandas','pyarrow','fastparquet','hydra','omegaconf','transformers','termcolor','sympy']
for m in mods:
    try:
        mod=__import__(m)
        print(m, 'OK', getattr(mod,'__version__','?'))
    except Exception as e:
        print(m, 'MISSING/ERR', type(e).__name__, e)
PY
```

Observed:

```text
numpy MISSING/ERR ModuleNotFoundError No module named 'numpy'
pandas MISSING/ERR ModuleNotFoundError No module named 'pandas'
pyarrow MISSING/ERR ModuleNotFoundError No module named 'pyarrow'
fastparquet MISSING/ERR ModuleNotFoundError No module named 'fastparquet'
hydra MISSING/ERR ModuleNotFoundError No module named 'hydra'
omegaconf MISSING/ERR ModuleNotFoundError No module named 'omegaconf'
transformers MISSING/ERR ModuleNotFoundError No module named 'transformers'
termcolor MISSING/ERR ModuleNotFoundError No module named 'termcolor'
sympy MISSING/ERR ModuleNotFoundError No module named 'sympy'
```

Command:

```bash
for f in papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/data/*; do
  printf '\n### %s\n' "$f"
  sed -n '1,20p' "$f"
done
```

Observed:

```text
### .../data/aime_2025.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:19df862b0e0c8cb826c6637a32a4d3c914019af009ed033493e3003a08a2a35f
size 17846

### .../data/hmmt_feb_2025.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:d4dd4ed78f0d97daf811cf7ab63827cbfc8d6105cc37eaed95fc2c5847503f85
size 12774

### .../data/test_code_contests.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:de56a100ec98584b4988ff590f5370d36805c7038646d03b9b6e16d42b5c2f6b
size 17278656

### .../data/test_livecodebench_v6.parquet
version https://git-lfs.github.com/spec/v1
oid sha256:645f8b8e4271151cd43206e91919164c58b9a68b532a2ce86dd2cf4f7dac8e49
size 118037264
```

The local `data/*.parquet` files are Git LFS pointer files, not readable parquet payloads. `git lfs` is also unavailable in this environment:

```bash
git -C papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification lfs ls-files
```

Observed:

```text
git: 'lfs' is not a git command.
```

The run scripts and `modal_eval.py` contain a HuggingFace fallback for missing/pointer datasets, but that requires extra packages and, for full reproduction, substantial model-serving infrastructure. In `eval/scripts/run_e2e_pairwise.sh`, lines 94-101 auto-download datasets through `huggingface_hub` when files are missing or smaller than 10KB. In `modal_eval.py`, lines 116-139 pull LFS files and fallback-download from `harman/v1-verification-datasets`.

### Code-level trace of the core mechanism

Pairwise aggregation:

- `eval/verify_utils.py` lines 123-138 clamp pair ratings and derive the winner.
- `eval/verify_utils.py` lines 141-158 implement the paper's weighted win-rate formula: `w=max(abs(rA-rB)/9, tau)`, weighted wins, total weights, and `mu=wins/W`.

Swiss tournament:

- `eval/verify_pairwise.py` lines 477-496 initialize `rank_swiss`.
- Lines 555-589 perform an initial random disjoint pairing round.
- Lines 607-671 enforce minimum-degree coverage.
- Lines 688-759 perform Swiss refinement, sorting by current `mu`, pairing nearby candidates within a window, and preferring unseen pairs.

Pointwise baseline:

- `eval/verify_pointwise.py` lines 170-202 score each solution independently and rank by descending scalar score. There is no secondary randomization or tie correction; if all scores are saturated, Python's stable sort preserves the original candidate order.

### Synthetic sanity check

Command:

```bash
python - <<'PY'
# Dependency-free reproduction of the paper/repo's aggregation formula and default Swiss pairing control.
# Mirrors eval/verify_utils.py:_update_pairwise_aggregates/_mu and verify_pairwise.py:rank_swiss.
import random

def sanitize(rA, rB):
    rA=max(1,min(10,int(rA))); rB=max(1,min(10,int(rB)))
    return ('A' if rA>rB else 'B' if rB>rA else 'tie'), rA, rB

def update(i,j,winner,rA,rB,wins,W,rating_sum,rating_cnt,tau=0.1):
    margin=abs(rA-rB)/9.0
    w=max(margin,tau)
    p=0.5 if winner=='tie' else (1.0 if winner=='A' else 0.0)
    wins[i]+=w*p; wins[j]+=w*(1.0-p)
    W[i]+=w; W[j]+=w
    rating_sum[i]+=rA; rating_cnt[i]+=1
    rating_sum[j]+=rB; rating_cnt[j]+=1

def mu(i,wins,W): return wins[i]/W[i] if W[i]>0 else 0.5
def avg(i,rating_sum,rating_cnt): return rating_sum[i]/rating_cnt[i] if rating_cnt[i]>0 else 0.0

correct=[0,1,0,1,0,1]
pointwise_scores=[10,10,10,10,10,10]

def judge_pair(i,j):
    if correct[i] and not correct[j]: return sanitize(10,3)
    if correct[j] and not correct[i]: return sanitize(3,10)
    if correct[i] and correct[j]: return sanitize(9,9)
    return sanitize(4,4)

def rank_swiss(n,budget,seed=0,min_degree=2,max_window=8,tau=0.1):
    rng=random.Random(seed)
    wins=[0.0]*n; W=[0.0]*n; rating_sum=[0]*n; rating_cnt=[0]*n
    counts=[[0]*n for _ in range(n)]; pairs=[]; used=0
    idx=list(range(n)); rng.shuffle(idx)
    for i,j in [(idx[k],idx[k+1]) for k in range(0,n-1,2)][:budget-used]:
        w,rA,rB=judge_pair(i,j); update(i,j,w,rA,rB,wins,W,rating_sum,rating_cnt,tau)
        counts[i][j]+=1; counts[j][i]+=1; pairs.append((i,j,w,rA,rB)); used+=1
    def mus(): return [mu(i,wins,W) for i in range(n)]
    mu_list=mus(); degrees=[sum(1 for jj in range(n) if counts[ii][jj]>0) for ii in range(n)]
    while used<budget and any(d<min_degree for d in degrees):
        needers=[ii for ii in range(n) if degrees[ii]<min_degree]
        needers.sort(key=lambda ii: degrees[ii])
        batch=[]; used_in_batch=set()
        for i in needers:
            if i in used_in_batch or used+len(batch)>=budget: continue
            candidates=[jj for jj in range(n) if jj!=i and jj not in used_in_batch and counts[i][jj]==0]
            if not candidates:
                degrees[i]=min_degree; continue
            j=min(candidates,key=lambda jj:(abs(mu_list[i]-mu_list[jj]),degrees[jj]))
            batch.append((i,j)); used_in_batch.add(i); used_in_batch.add(j)
        if not batch: break
        for i,j in batch:
            w,rA,rB=judge_pair(i,j); update(i,j,w,rA,rB,wins,W,rating_sum,rating_cnt,tau)
            counts[i][j]+=1; counts[j][i]+=1; pairs.append((i,j,w,rA,rB)); used+=1; degrees[i]+=1; degrees[j]+=1
        mu_list=mus()
    while used<budget and n>2:
        order=sorted(range(n), key=lambda t:(-mu(t,wins,W),-avg(t,rating_sum,rating_cnt)))
        mu_list=mus(); used_in_round=set(); batch=[]
        k=0
        while k<n:
            i=order[k]
            if i in used_in_round: k+=1; continue
            candidates=[]
            for t in range(k+1, min(k+1+max_window,n)):
                j=order[t]
                if j in used_in_round: continue
                candidates.append(((0 if counts[i][j]==0 else 1, abs(mu_list[i]-mu_list[j]), counts[i][j]), j))
            if candidates:
                candidates.sort(key=lambda x:x[0]); j=candidates[0][1]
                batch.append((i,j)); used_in_round.add(i); used_in_round.add(j)
            k+=1
        if not batch: break
        for i,j in batch[:budget-used]:
            w,rA,rB=judge_pair(i,j); update(i,j,w,rA,rB,wins,W,rating_sum,rating_cnt,tau)
            counts[i][j]+=1; counts[j][i]+=1; pairs.append((i,j,w,rA,rB)); used+=1
    order=sorted(range(n), key=lambda t:(-mu(t,wins,W),-avg(t,rating_sum,rating_cnt)))
    return order, [round(mu(i,wins,W),3) for i in range(n)], pairs

point_order=sorted(range(len(correct)), key=lambda i:-pointwise_scores[i])
pair_order, pair_mu, pairs=rank_swiss(len(correct), budget=3*len(correct), seed=0)
print('correctness:', correct)
print('pointwise_scores:', pointwise_scores)
print('pointwise_order:', point_order, 'selected:', point_order[0], 'selected_correct:', bool(correct[point_order[0]]))
print('pairwise_pairs_used:', len(pairs), 'first_8_pairs:', pairs[:8])
print('pairwise_mu:', pair_mu)
print('pairwise_order:', pair_order, 'selected:', pair_order[0], 'selected_correct:', bool(correct[pair_order[0]]))
print('support_status: algorithmic sanity check supports selection improvement when pairwise ratings carry comparative signal; it does not reproduce paper benchmark accuracies.')
PY
```

Observed:

```text
correctness: [0, 1, 0, 1, 0, 1]
pointwise_scores: [10, 10, 10, 10, 10, 10]
pointwise_order: [0, 1, 2, 3, 4, 5] selected: 0 selected_correct: False
pairwise_pairs_used: 18 first_8_pairs: [(4, 2, 'tie', 4, 4), (1, 0, 'A', 10, 3), (5, 3, 'tie', 9, 9), (0, 2, 'tie', 4, 4), (1, 3, 'tie', 9, 9), (4, 5, 'B', 3, 10), (1, 5, 'tie', 9, 9), (3, 2, 'A', 10, 3)]
pairwise_mu: [0.057, 0.943, 0.057, 0.943, 0.102, 0.898]
pairwise_order: [1, 3, 5, 4, 0, 2] selected: 1 selected_correct: True
support_status: algorithmic sanity check supports selection improvement when pairwise ratings carry comparative signal; it does not reproduce paper benchmark accuracies.
```

Interpretation: the released weighted pairwise/Swiss ranking logic can select a correct candidate in a setting where pointwise scoring saturates and preserves an incorrect first candidate. This supports the plausibility of the mechanism described in the paper. It does not validate the reported benchmark percentages, scaling curves, trained PairRL claims, or RSA/SWE-bench comparisons.

## Reproduction Outcome

Outcome: partial support, with benchmark reproduction blocked.

Evidence supporting the claim:

- The released `V_1-Infer` implementation matches the paper's stated weighted pairwise aggregation and Swiss refinement structure.
- The pointwise baseline implementation is a strict independent-score selector, so score saturation can directly cause the failure mode described in the paper.
- A dependency-free synthetic check using the repo's formulas and default pairing strategy reproduced the expected qualitative behavior: pairwise comparisons recovered a correct candidate while pointwise saturated scores selected an incorrect one.

Evidence not reproduced:

- I did not reproduce any reported benchmark number such as 73.33% on CodeContests, 76.3% on LiveCodeBench-v6, 86.7% on AIME 2025, or the PairRL gains in Section 5.
- I did not reproduce the training claim that unified PairRL co-training improves generation quality and self-verification, because I found no training code, no trained checkpoints, and no raw result artifacts in the official repo checkout.
- I did not run the full evaluation pipeline because the local data files are Git LFS pointers, `git lfs` is unavailable, core dependencies are absent, and the paper-scale runs require large model serving infrastructure. The README itself states the expected results used 3x H100 GPUs via Modal.

## Blockers and Hidden Assumptions

1. The `data/*.parquet` files in the local official repo checkout are Git LFS pointer files, not actual datasets.
2. The environment lacks required packages listed in the README installation command, including `numpy`, `polars`, `hydra-core`, `transformers`, `sglang`, and `sympy`.
3. The released repository does not appear to include cached generations, result parquets, trained PairRL checkpoints, training scripts, or raw numeric logs behind the paper figures.
4. Full reproduction depends on external model availability, GPU resources, SGLang/Modal setup, and HuggingFace dataset downloads.
5. The synthetic check establishes that the algorithm can exploit better pairwise comparative signal; it cannot establish that the target LLMs actually provide that signal at the reported rates.

## Match / Partial Match / Mismatch / Blocked

Result: partial match.

The code-level mechanism and synthetic sanity check support the paper's qualitative explanation for why pairwise verification can outperform pointwise scoring. The empirical benchmark claim remains unreproduced from the provided local artifacts. The unified generation-plus-verification training claim is especially weakly reproducible from this repository because the necessary training and checkpoint artifacts are absent.

## Agreement With Reproducer A

Not assessed. The task explicitly instructed me not to read Independent Reproducer A's report, so I did not compare outcomes. After both passes are complete, a consolidator should compare this report against Reproducer A's findings.

## Score Impact

Reproducibility impact: materially negative for the full paper, despite partial support for the inference mechanism.

I would credit the paper for releasing readable inference code whose core aggregation logic matches the described method and passes a targeted sanity check. I would not credit the paper as independently reproducing its central empirical results from the provided artifact state. The benchmark and PairRL claims require substantial unprovided or external resources: actual LFS datasets, dependencies, model serving, generated candidates, trained checkpoints, and raw result logs.
