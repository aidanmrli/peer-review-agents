# Independent Reproducer A Report

Paper: `59386b0e-204c-4c09-986a-109be4967508`, "Graph-GRPO: Training Graph Flow Models with Reinforcement Learning"

Role: Independent Reproducer A, starting from the paper text and official artifacts.

Date: 2026-04-24

## Claim Attempted

I attempted the smallest meaningful reproduction of the central empirical claim:

> Graph-GRPO, implemented as GRPO/RL fine-tuning plus refinement on a DeFoG graph flow backbone, materially improves DeFoG on synthetic graph generation and molecular optimization, e.g. Tree V.U.N. from 73.5% to 97.5%, PMO AUC-top10 from 11.079 for DeFoG to 17.450 with RL and 18.987 with refinement, and stronger docking hit ratios.

The intended lightweight reproduction target was not full training. The intended minimum was to locate and invoke a Graph-GRPO rollout/training/refinement code path, config, checkpoint, generated-sample file, or metric script sufficient to reproduce one small reported result or at least verify the claimed RL/refinement machinery.

## Setup Used

Workspace:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent2
```

Artifacts inspected:

```bash
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/paper.pdf
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/main.tex
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/ref.bib
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG
```

Repository state:

```bash
git -C papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG rev-parse HEAD
# 365bda9affadd5c2307014a0532ddaa244399441

git -C papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG remote -v
# origin https://github.com/manuelmlmadeira/DeFoG.git
```

Local Python environment:

```bash
python --version
# Python 3.12.12

python -c "import torch; print('torch', torch.__version__)"
# ModuleNotFoundError: No module named 'torch'
```

## Commands and Observations

### 1. Artifact inventory

Command:

```bash
find papers/59386b0e-204c-4c09-986a-109be4967508 -maxdepth 3 -type f | sort
```

Observed: the top-level paper artifact contains the LaTeX, PDF, figures, and a clone of `DeFoG`. No local `.ckpt`, `.pt`, `.pth`, generated sample `.pkl`, experiment result `.csv`, or Graph-GRPO-specific script appeared in the top-level artifact listing.

Targeted checkpoint/config search:

```bash
find papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG -type f \
  \( -name '*.ckpt' -o -name '*.pt' -o -name '*.pth' -o -name '*.pkl' -o -name '*.csv' -o -name '*.json' -o -name '*.yaml' -o -name '*.yml' -o -name '*.sh' \) | sort
```

Observed: only DeFoG configs, Docker/environment files, and dataset helper references were present. There were no local Graph-GRPO checkpoints, generated samples, RL configs, PMO configs, docking configs, refinement configs, or runnable experiment scripts.

### 2. Search for Graph-GRPO/RL/refinement implementation

Command:

```bash
rg -n "GRPO|Graph-GRPO|reinforcement|reward|policy|advantage|rollout|refinement|priority|oracle|PMO|docking|valsartan|scaffold|RL|ppo|kl|trajectory|buffer|prescreen" \
  papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG
```

Observed result: no Graph-GRPO implementation code was found. The only semantically relevant matches were in `README.md` for generic DeFoG usage/checkpoints, in `graph_discrete_flow_model.py` for sample serialization, and in unrelated metric/helper text. A file-path-only grep for `grpo|rl|ppo|reward|oracle|dock|pmo|refin|screen|buffer|checkpoint|ckpt` similarly found only Docker/dependency paths, not method code.

This is a central blocker: the linked repository is the base DeFoG implementation, not an implementation of Graph-GRPO.

### 3. Inspect DeFoG rate matrix path

Paper claim location:

- `main.tex` lines 491-508 define the claimed analytic rate matrix for Graph-GRPO and state that it enables policy-gradient optimization.
- `main.tex` lines 536-604 describe rollout collection, reward evaluation, GRPO objective, advantage normalization, importance ratios, and KL regularization.

Code inspected:

- `artifacts/DeFoG/src/flow_matching/rate_matrix.py` lines 25-73 define `compute_graph_rate_matrix`.
- `artifacts/DeFoG/src/flow_matching/rate_matrix.py` lines 32-42 sample a pseudo clean graph with `flow_matching_utils.sample_discrete_features(...)`, then pass the sampled labels into `compute_dfm_variables`.
- `artifacts/DeFoG/src/flow_matching/rate_matrix.py` lines 75-112 compute conditional DFM variables from the sampled `X_1_sampled` and `E_1_sampled`.
- `artifacts/DeFoG/src/graph_discrete_flow_model.py` lines 109-116 instantiate only the base `RateMatrixDesigner`.
- `artifacts/DeFoG/src/graph_discrete_flow_model.py` lines 548-636 implement the DeFoG sampling loop, not a rollout cache, GRPO loss, reward feedback, or refinement loop.

Observed result: the provided code path remains Monte Carlo/pseudo-sample based. I did not find code implementing the paper's analytic Eq. ARM over the full predicted distribution, the GRPO objective, the old/new policy ratio, advantage computation, reward model/oracle, KL-to-reference policy, dynamic priority buffer, adaptive prior update, or refinement loop.

### 4. Documented quick-start execution

The DeFoG README says `main.py` is inside `src` and gives:

```bash
python main.py +experiment=debug
```

I ran:

```bash
cd papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG/src
python main.py +experiment=debug
```

Observed result:

```text
Traceback (most recent call last):
  File ".../artifacts/DeFoG/src/main.py", line 5, in <module>
    import graph_tool
ModuleNotFoundError: No module named 'graph_tool'
```

Even the base DeFoG debug run is not executable in the current environment without installing the specified environment. More importantly, this command would only test DeFoG, not Graph-GRPO. I did not attempt environment creation or training because the requested scope was lightweight checks, and the paper itself reports requiring substantial compute.

### 5. Compute and reproducibility requirements from the paper

The paper's own stated resource requirements are non-lightweight:

- `main.tex` lines 1424-1425: one NVIDIA RTX PRO 6000 GPU with 96GB VRAM and 22 CPU cores; approximately 20 hours for each molecular task and 5 hours for synthetic graph generation.
- `main.tex` lines 1451-1454: GRPO group size `K=60`, asymmetric PPO clipping, advantage clipping, and KL penalty.
- `main.tex` lines 1457-1464: seed counts and dynamic prior update details.
- `main.tex` lines 1458-1460 contain a duplicated/or grammatically inconsistent description of the first 300 oracle calls for refinement initialization, but the high-level budget schedule is recoverable.

This makes full empirical reproduction impossible under the current role's lightweight constraint even if code were provided.

### 6. Table/math sanity checks

I checked internal arithmetic for the PMO AUC-top10 table and ablation deltas using only the reported table values.

Command:

```bash
python - <<'PY'
prescreen=[0.994,0.823,0.890,0.942,0.995,0.984,0.948,0.995,0.932,0.910,0.388,0.322,0.980,0.924,0.690,0.944,0.928,0.711,0.879,0.842,0.711,0.841,0.697]
cold=[0.994,0.823,0.890,0.762,0.995,0.984,0.965,0.995,0.932,0.910,0.388,0.300,0.974,0.922,0.689,0.944,0.928,0.622,0.879,0.842,0.711,0.841,0.697]
print('prescreen sum rounded 3 =', round(sum(prescreen), 3), 'raw', sum(prescreen))
print('cold-start sum rounded 3 =', round(sum(cold), 3), 'raw', sum(cold))
print('ablation deltas: RL-base', round(17.450-11.079,3), 'refinement-RL', round(18.987-17.450,3), 'prescreen-refinement', round(19.270-18.987,3))
PY
```

Observed:

```text
prescreen sum rounded 3 = 19.27 raw 19.27
cold-start sum rounded 3 = 18.987 raw 18.987
ablation deltas: RL-base 6.371 refinement-RL 1.537 prescreen-refinement 0.283
```

Result: the PMO sum rows and ablation deltas are arithmetically consistent with the values printed in the paper. This is only a table consistency check, not an empirical reproduction.

## Reproduction Outcome

Outcome: **blocked / weak reproducibility**.

I could not reproduce the central Graph-GRPO empirical claim from the provided artifacts. The official linked repository clone is DeFoG at commit `365bda9affadd5c2307014a0532ddaa244399441`; it contains base DeFoG training/sampling code and configs, but I found no Graph-GRPO/RL/refinement implementation, no Graph-GRPO experiment configs, no reward/oracle integration, no PMO/docking scripts, no local checkpoints, and no generated samples/results files that would allow a lightweight verification of any reported Graph-GRPO metric.

The smallest executable check, the README's DeFoG debug command, fails in the current environment because `graph_tool` is missing. A separate Python import check also shows `torch` is missing. These environment blockers are secondary to the main artifact blocker: even a working DeFoG environment would not expose the Graph-GRPO method claimed in the paper.

## Decision-Relevant Consequence

The paper's empirical acceptance case rests on RL fine-tuning and refinement improving a DeFoG backbone. In the provided official artifacts, that mechanism is not independently reproducible by this role. The table values are internally consistent, but the code/artifact release does not support verifying the central claimed improvement, the GRPO objective implementation, reward handling, dynamic prior update, or refinement budget accounting. Under a reproducibility-first standard, this should be treated as a material weakness unless another role finds a separate official Graph-GRPO implementation or sufficient checkpoints/generated outputs.
