# Independent Reproducer A Report

Paper ID: `c993ba35-65e0-4290-a66a-c128e33410f4`

Title: "Learning Approximate Nash Equilibria in Cooperative Multi-Agent Reinforcement Learning via Mean-Field Subsampling"

Assigned role: Independent Reproducer A

Task scope: Reproduce the smallest meaningful empirical unit from the official artifacts and linked repository, starting from the documented repository command. This pass focused on whether a reviewer can recover the paper's reported numerical trend that larger subsampling budgets `k` improve discounted reward, and whether the public code can be used as evidence for the paper's claimed `ALTERNATING-MARL` algorithm.

## Evidence Examined

- Official artifact source and PDF under `papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/`.
- Linked repository clone at `papers/c993ba35-65e0-4290-a66a-c128e33410f4/repos/alternating-marl`, commit `2b1a57e151ac57d77fa8bbadbbd1de80a692320e`.
- Repository README and primary scripts:
  - `README.md`
  - `requirements.txt`
  - `scripts/marl_example.py`
  - `scripts/alternating_marl.py`
  - `scripts/hyperparameters.json`
- Paper experiment section in `artifacts/main.tex:1534-1625`.

## Commands and Environment

Base environment:

```bash
python3 --version
# Python 3.12.12

python3 scripts/marl_example.py
# Traceback ... ModuleNotFoundError: No module named 'numpy'
```

Isolated environment created inside the cloned repository:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python --version
# Python 3.12.12

.venv/bin/python - <<'PY'
import numpy, matplotlib, seaborn, torch
print('numpy', numpy.__version__)
print('matplotlib', matplotlib.__version__)
print('seaborn', seaborn.__version__)
print('torch', torch.__version__)
PY
# numpy 2.4.4
# matplotlib 3.10.9
# seaborn 0.13.2
# torch 2.11.0+cu130
```

Full documented driver:

```bash
timeout 180 .venv/bin/python scripts/marl_example.py
# exited with code 124 after 180 seconds and emitted no buffered progress/output
```

Reduced reproduction using the public API:

```bash
PYTHONPATH=scripts .venv/bin/python - <<'PY'
import time
from marl_example import run_single_k
for k in [1, 5, 10]:
    t = time.time()
    res = run_single_k(k, n_eval_seeds=2, verbose=False)
    print(f"k={k} mean={res['value_mean']:.4f} std={res['value_std']:.4f} train={res['train_time']:.3f} wall={time.time()-t:.3f}")
PY
```

Observed output:

```text
k=1 mean=83.1134 std=0.1055 train=0.723 wall=1.733
k=5 mean=85.7163 std=0.9774 train=0.771 wall=1.784
k=10 mean=87.2679 std=1.5376 train=0.458 wall=1.471
```

## Findings

The default documented command is not reproducible in the base reviewer environment because required packages are not installed. After installing the unpinned requirements in a local venv, the full `scripts/marl_example.py` driver did not produce a result within a 180 second cap. This blocks direct reproduction of the paper's full `k=1..50`, 15-seed numerical figure from a clean command.

A reduced run using the repository's `run_single_k` API does show the qualitative local trend `k=1 < k=5 < k=10` under two evaluation seeds. This is a partial empirical sanity check for the idea that larger subsamples can improve the toy warehouse reward.

The reduced run does not reproduce the paper's stated experiment. The paper reports 15 seeds per `k`, horizon 100, `N_steps=10`, and `k=1..50` in `artifacts/main.tex:1578-1602`; the repository config uses `n_outer_iterations=5` and evaluation horizon 50 in `scripts/hyperparameters.json:21-28`. The public full driver also writes figure names `reward_vs_k.png` and `runtime_vs_k.png`, whereas the LaTeX includes `sections/Figures/k_vs_n_rewards.png` and `sections/Figures/k_vs_n_runtimes.png`.

Most importantly, the executable code does not implement the theoretical algorithm whose claim I attempted to reproduce. The global optimizer uses deterministic rounded expected count transitions rather than the empirical Bellman operator with `m` samples, and the local optimizer uses a reduced `(s_g,s_l)` value iteration rather than the paper's chained MDP plus UCFH. Therefore the successful reduced trend is evidence for a toy heuristic, not for the theorem that `ALTERNATING-MARL` learns a `\tilde O(1/\sqrt{k})` approximate Nash equilibrium.

## Reproduction Outcome

Outcome: partial match for a weak qualitative empirical trend; blocked/mismatch for the paper's central reproducibility target.

- Match: a small public-code run showed increasing reward for `k=1,5,10`.
- Blocked: the documented full driver did not finish within 180 seconds after dependencies were installed and emitted no progress.
- Mismatch: the released settings and algorithm differ from the paper's reported configuration and proof-level `G-LEARN`/`L-LEARN`/`UPDATE` procedures.

## Limitations and Blockers

- I did not alter repository code or hyperparameters.
- I did not regenerate the paper's included PNGs because the full driver timed out and because the script names do not directly match the LaTeX figure names.
- The reproduction uses unpinned package versions from `requirements.txt`, so exact numerical repeatability is not guaranteed.
- Two earlier subagent attempts for this role were interrupted or became unavailable; this report is the completed local independent pass.

## Confidence Level

High for the execution and configuration findings. Medium for the empirical trend because the reduced run used only three `k` values and two evaluation seeds.

## Decision Impact

The paper receives only limited empirical reproducibility credit. A small toy k-sweep can be recovered after manual environment setup, but the full reported experiment is not a one-command reproduction and the code does not reproduce the theoretical algorithm. This materially weakens confidence in the paper's empirical validation and does not independently support the approximate Nash equilibrium theorem.
