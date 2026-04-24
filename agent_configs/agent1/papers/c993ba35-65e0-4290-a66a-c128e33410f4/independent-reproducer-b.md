# Independent Reproducer B Report

Paper ID: `c993ba35-65e0-4290-a66a-c128e33410f4`

Title: "Learning Approximate Nash Equilibria in Cooperative Multi-Agent Reinforcement Learning via Mean-Field Subsampling"

Assigned role: Independent Reproducer B

Task scope: Independently validate the core claim through a code/configuration trace rather than rerunning the same full experiment. This pass checked whether the public implementation and artifact configuration match the paper's stated numerical setup and proof-level algorithms.

## Evidence Examined

- Paper source:
  - `artifacts/sections/preliminaries.tex:147-324` for `G-LEARN`, `L-LEARN`, `ALTERNATING-MARL`, and online execution.
  - `artifacts/main.tex:1534-1625` for numerical setup and reported figures.
- Repository clone:
  - `scripts/marl_example.py`
  - `scripts/hyperparameters.json`
  - `scripts/alternating_marl.py`
  - `scripts/global_agent_optimizer.py`
  - `scripts/local_agent_optimizer.py`
  - `scripts/plot_simulation.py`
- Repository commit: `2b1a57e151ac57d77fa8bbadbbd1de80a692320e`.

## Commands and Trace Steps

```bash
nl -ba scripts/marl_example.py | sed -n '70,130p'
nl -ba scripts/hyperparameters.json
nl -ba scripts/alternating_marl.py | sed -n '260,325p'
nl -ba scripts/global_agent_optimizer.py | sed -n '50,195p'
nl -ba scripts/local_agent_optimizer.py | sed -n '1,140p'
nl -ba scripts/plot_simulation.py | sed -n '250,270p'
sed -n '1534,1625p' artifacts/main.tex
```

## Findings

The paper's numerical setup is not reproduced by the released code as written.

First, the global reward differs. The paper states in `artifacts/main.tex:1563-1571` that

```tex
r_g(s_g, a_g) = 4 - |s_g - a_g|
```

but `scripts/marl_example.py:77-87` sets the entire global reward table to `1.0`. This is a direct environment mismatch, not a random-seed issue.

Second, the reported hyperparameters differ. The paper table gives `N_steps=10`, `15 seeds per k`, `50 rollouts per seed`, and horizon `100` (`artifacts/main.tex:1578-1602`). The repository config uses `n_outer_iterations=5` and evaluation horizon `50` (`scripts/hyperparameters.json:21-28`). The training loop also evaluates convergence using only 20 rollouts of horizon 30 (`scripts/alternating_marl.py:290-291`).

Third, the figure-generation path does not exactly match the paper figures. The paper includes `k_vs_n_rewards.png`, `k_vs_n_runtimes.png`, and a heatmap caption for `k=1,10,20,35` (`artifacts/main.tex:1608-1618`). The main script in `marl_example.py` writes `reward_vs_k.png` and `runtime_vs_k.png`; `plot_simulation.py` has a default list including `20`, but its `__main__` call uses `[1, 10, 35]` at `scripts/plot_simulation.py:262-268`.

Fourth, the algorithmic code path is not the proof-level algorithm. In `scripts/global_agent_optimizer.py`, the `n_mc` parameter is stored at lines 76-85 but the Bellman backup uses deterministic rounded expected count transitions (`global_agent_optimizer.py:160-188`). This is not the empirical Bellman sampling described for `G-LEARN` in `artifacts/sections/preliminaries.tex:180-233`.

Fifth, `scripts/local_agent_optimizer.py` does not implement the `k`-chained or mean-field chained MDP and does not run UCFH. It estimates an effective global transition from multinomial samples around a stationary mean-field distribution and solves a reduced `(s_g,s_l)` value iteration. This is not the local-agent procedure described in `artifacts/sections/preliminaries.tex:236-264`.

Sixth, `scripts/alternating_marl.py:281-320` runs `g_learn()` and `l_learn()` back-to-back and then uses a scalar rollout estimate for best-value reversion and convergence. It does not implement the formal `UPDATE` rule from `artifacts/sections/preliminaries.tex:281-303`, which requires all-state value interval comparisons with tolerance `2 eta`.

## Independent Result

Outcome: mismatch for the central implementation claim.

This pass did not need the same reduced execution route as Reproducer A to identify the mismatch. The code and paper are inconsistent at the level of reward definition, horizon/iteration settings, figure-generation pathway, and the algorithms used for global learning, local learning, and termination. Even if the repository produces increasing rewards as `k` increases, it is not a reproduction of the theoretical `ALTERNATING-MARL` algorithm or the exact paper experiment.

## Agreement With Reproducer A

This pass agrees with Reproducer A's conclusion after both passes: the artifacts provide partial support for a qualitative toy simulation trend, but they do not independently reproduce the paper's central empirical/theoretical claim. Reproducer A reached this through execution behavior; this pass reached it through source and configuration consistency.

## Limitations and Blockers

- I did not run the full k-sweep because this role intentionally used an independent static/trace route.
- I did not inspect external sources or use future signals.
- I did not modify the authors' code.
- Two earlier subagent attempts for this role were interrupted or became unavailable; this report is the completed local independent pass.

## Confidence Level

High. The mismatches are direct source-level comparisons between the paper and repository.

## Decision Impact

The implementation cannot be credited as a faithful reproduction of the central algorithm. It supports, at most, a simplified illustrative simulator. For a reproducibility-first review, this is a major negative finding and should materially lower confidence in the empirical validation and implementation-backed claims.
