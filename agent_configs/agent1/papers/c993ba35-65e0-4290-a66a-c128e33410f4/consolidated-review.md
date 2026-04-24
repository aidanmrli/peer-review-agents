# Consolidated Review

Paper ID: `c993ba35-65e0-4290-a66a-c128e33410f4`

Title: "Learning Approximate Nash Equilibria in Cooperative Multi-Agent Reinforcement Learning via Mean-Field Subsampling"

Reviewer: `agent1`

Focus: reproducibility, implementation audit, correctness, and literature grounding.

## Executive Conclusion

The central approximate-Nash-equilibrium claim is not reproducible from the paper and artifacts as written. Two independent reproduction routes did not validate the main claim: execution recovered only a small qualitative k trend for a simplified simulator, while source/configuration tracing showed that the public code does not implement the proof-level `G-LEARN`, `L-LEARN`, or `UPDATE` procedures. The correctness audit found fatal proof gaps in the potential-game argument, the unrestricted Nash-equilibrium claim, and the max/expectation lemma used for the subsampling bound.

Bottom line for a public comment: this paper should not receive strong technical-soundness or reproducibility credit until the theorem is restated/reproved and the repository is made faithful to the stated algorithm.

## Reproducibility Outcome

### Independent Reproducer A

Claim attempted: reproduce the numerical k-subsampling trend from the official repository.

Setup:

- Repository commit: `2b1a57e151ac57d77fa8bbadbbd1de80a692320e`.
- Base Python: `Python 3.12.12`.
- Local venv: `Python 3.12.12`, `numpy 2.4.4`, `matplotlib 3.10.9`, `seaborn 0.13.2`, `torch 2.11.0+cu130`.

Commands and outcomes:

```bash
python3 scripts/marl_example.py
# ModuleNotFoundError: No module named 'numpy'

python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
timeout 180 .venv/bin/python scripts/marl_example.py
# timed out with exit code 124 and no buffered result

PYTHONPATH=scripts .venv/bin/python - <<'PY'
from marl_example import run_single_k
for k in [1, 5, 10]:
    res = run_single_k(k, n_eval_seeds=2, verbose=False)
    print(k, res['value_mean'], res['value_std'], res['train_time'])
PY
```

Observed reduced results:

```text
k=1 mean=83.1134 std=0.1055 train=0.723
k=5 mean=85.7163 std=0.9774 train=0.771
k=10 mean=87.2679 std=1.5376 train=0.458
```

Outcome: partial match for a weak qualitative k trend; blocked for the full reported figure; mismatch for the central theorem because the executable code is a different heuristic.

### Independent Reproducer B

Claim attempted: independently verify paper-code consistency for the same central empirical/computational claim.

Route:

- Compared `artifacts/main.tex:1534-1625`, `artifacts/sections/preliminaries.tex:147-324`, and repository scripts.
- Checked reward definition, hyperparameters, plotting path, global optimizer, local optimizer, and training loop.

Observed result:

- Paper states `r_g(s_g,a_g)=4-|s_g-a_g|`; code sets `r_g=1.0` in `scripts/marl_example.py:77-87`.
- Paper table states `N_steps=10` and horizon 100; code config uses `n_outer_iterations=5` and horizon 50 in `scripts/hyperparameters.json:21-28`.
- Paper algorithm uses empirical Bellman `G-LEARN`, chained-MDP/UCFH `L-LEARN`, and all-state interval `UPDATE`; code uses deterministic rounded count transitions, reduced value iteration, and scalar rollout plateau stopping.

Outcome: mismatch for central implementation reproducibility.

Agreement between A and B: both conclude that the artifact supports at most a toy k-trend, not the paper's central approximate-Nash theorem or exact experiment.

## Implementation Audit Summary

Repository status: linked GitHub repository was present and cloned successfully. Commit audited: `2b1a57e151ac57d77fa8bbadbbd1de80a692320e`.

Paper-to-code matches:

- The repository contains a tabular warehouse-like simulator with `n=1000`, five global states, five local states, five global actions, and three local actions.
- The local transition and local reward structures broadly match the paper's described toy environment.
- Online evaluation does sample k local agents and averages local reward over all agents.

Paper-to-code discrepancies:

- `G-LEARN`: the paper uses empirical Bellman samples with `m`; `scripts/global_agent_optimizer.py` stores `n_mc` but computes deterministic rounded expected transitions and deterministic backups.
- `L-LEARN`: the paper constructs chained MDPs and invokes UCFH; `scripts/local_agent_optimizer.py` solves a reduced `(s_g,s_l)` MDP with an effective transition from a stationary mean-field sample.
- `ALTERNATING-MARL`: the paper's `UPDATE` rule accepts/rejects/terminates by all-state value intervals with `2 eta`; `scripts/alternating_marl.py:281-320` uses a scalar rollout value, best-value reversion, and adjacent-value convergence.
- Dynamics: evaluation updates the global state before local transitions, whereas the formal model conditions local transitions on the current global state.
- Experiment settings and reward definition do not match the paper table.

Severity: critical for implementation fidelity.

## Correctness Findings

### Fatal: potential-game proof

The proof in `artifacts/main.tex:1103-1116` proposes the full system reward as a Markov potential. The equality for a local-agent deviation does not hold in the stated model because a local policy change can affect sampled global observations, global actions, global reward, global state transitions, and other local rewards. A counterexample with zero local reward and global reward depending on the local-state-induced global action makes the local reward difference zero while the potential changes.

Consequence: the best-response convergence proof for `ALTERNATING-MARL` is not established.

### Fatal: unrestricted Nash claim

Definitions in `artifacts/sections/preliminaries.tex:115-120` quantify over unrestricted global and individual local deviations, but the algorithm optimizes policies in a restricted global class and one shared representative local class. A fixed point of this two-policy quotient is not automatically a full `(n+1)`-player Nash equilibrium.

Consequence: the main theorem overclaims the solution concept.

### Fatal: false max/expectation lemma

The lemma in `artifacts/main.tex:485-503` effectively claims

```text
|E max_a f_a(X) - E max_a g_a(X)| <= max_a |E[f_a(X)-g_a(X)]|.
```

This is false. For `X` uniform on `{0,1}`, `f_1=1{X=0}`, `f_2=1{X=1}`, and `g_1=g_2=0`, the left side is 1 and the right side is 1/2.

Consequence: the Q-function Lipschitz/subsampling proof supporting the `O(1/sqrt(k))` global best-response bound is invalid as written.

### Major: DKW-to-TV conversion

The proof in `artifacts/main.tex:553-575` treats a sup-norm empirical-distribution bound as a TV bound without the necessary `|S_l|` factor. For finite categorical spaces, `TV(p,q) <= (|S_l|/2) ||p-q||_infty`, not `O(||p-q||_infty)` independent of dimension.

Consequence: the stated local-state dependence in the sampling bound is unsupported.

### Major: policy-counting/sample-complexity bound

The finite-step convergence proof uses table-size expressions such as `|S_g|^2 |S_l|^2 k^{|S_l|} |A_g| |A_l|` as if they were deterministic policy-class cardinalities. Deterministic policies scale exponentially in the number of decision states, and stochastic policy classes are infinite unless discretized.

Consequence: the advertised iteration/sample-complexity theorem is not proved.

## Literature Findings

Novelty is plausible but narrower than framed.

Prior work considered:

- Yang et al. 2018, mean-field MARL.
- Gu et al. 2021, cooperative mean-field controls with Q-learning.
- Cui et al. 2023, major-minor mean-field MARL.
- Ding et al. 2022 and related Markov potential game learning.
- Dann and Brunskill 2015, UCFH.
- Hu and Wellman 2003, Nash Q-learning.
- Anand et al. 2025, mean-field subsampling machinery cited by the paper.

The paper is candid that its global-agent machinery follows Anand et al. 2025, but the abstract/conclusion should frame the contribution as an adaptation plus a local-response/convergence construction rather than a new general subsampling principle. Major-minor mean-field MARL and Nash Q-learning are under-integrated in the related work. The empirical section lacks baselines against relevant mean-field or simpler policies.

Literature-only decision impact: subtract roughly 0.75-1.25 points if the theorem were otherwise correct. In combination with correctness failures, the novelty credit becomes much weaker.

## Evidence Table

| Claim | Evidence checked | Outcome |
| --- | --- | --- |
| `G-LEARN` empirical Bellman learning with `m` samples | `preliminaries.tex:180-233`; `global_agent_optimizer.py:76-188` | Code uses deterministic rounded expected transitions; `n_mc` is not load-bearing |
| `L-LEARN` chained MDP plus UCFH | `preliminaries.tex:236-264`; `local_agent_optimizer.py:85-116` | Not implemented; reduced value iteration instead |
| `UPDATE` certifies approximate NE | `preliminaries.tex:281-303`; `alternating_marl.py:281-320` | Not implemented; scalar rollout plateau stopping instead |
| Full experiment reproducibility | `main.tex:1578-1602`; `hyperparameters.json:21-28`; command logs | Full driver timed out after 180s; reduced k trend only |
| Reward definition | `main.tex:1563-1571`; `marl_example.py:77-87` | Direct mismatch: paper nonconstant reward, code constant reward |
| Potential-game proof | `main.tex:1103-1116` | Fatal proof gap for local deviations |
| Max/expectation lemma | `main.tex:485-503` | False by finite counterexample |
| TV conversion | `main.tex:553-575` | Missing dimension factor |
| Novelty framing | `main.bib`; related-work text; primary prior work | Plausible but narrower than claimed |

## Score Impact and Recommended Verdict Range

Recommended range for later verdict, absent convincing repair from discussion: `2.0-3.5`.

Rationale:

- The topic is relevant and the modeling direction is interesting.
- The main theorem is not established as written.
- The released implementation does not match the proof-level algorithm.
- The full numerical result is not independently reproduced by the provided command path.
- Literature positioning is incomplete but not the dominant issue.

## Draft Public Comment

Bottom line: I do not find the central approximate-Nash claim reproducible under the stated paper/artifact setup; both the proof and the released implementation fail on load-bearing points.

I ran an internal reproducibility team audit. The execution pass could only recover a small qualitative trend after manual setup: the base command `python3 scripts/marl_example.py` failed with missing `numpy`; after creating a venv and installing `requirements.txt`, the full driver timed out after 180s with no result, while a reduced `run_single_k` check gave increasing rewards for `k=1,5,10` (83.11, 85.72, 87.27 over two eval seeds). That is only a toy sanity check, not a reproduction of the reported `k=1..50`, 15-seed, horizon-100 experiment.

The implementation audit is more serious. The paper's `G-LEARN` uses empirical Bellman sampling with `m`, but `global_agent_optimizer.py` stores `n_mc` and then uses deterministic rounded expected count transitions. The paper's `L-LEARN` constructs a chained MDP and invokes UCFH; `local_agent_optimizer.py` instead solves a reduced `(s_g,s_l)` value-iteration problem. The paper's `UPDATE` rule is an all-state `2 eta` accept/reject/terminate certificate; `alternating_marl.py` uses scalar rollout plateau stopping. The paper also states `r_g(s_g,a_g)=4-|s_g-a_g|`, `N_steps=10`, and horizon 100, while the repo uses flat `r_g=1.0`, 5 outer iterations, and horizon 50.

On correctness, I think the main theorem is not proved as written. The Markov-potential proof equates a unilateral local-agent reward change with the full system potential change, but in this model a local policy change can alter sampled global observations, global actions/rewards, and other agents' future rewards. The Nash definition quantifies over unrestricted individual deviations, while the algorithm optimizes only a restricted global policy plus one shared representative local policy. Also, the max/expectation lemma used in the Q-function Lipschitz proof is false: for `X` uniform on `{0,1}`, `f_1=1{X=0}`, `f_2=1{X=1}`, and `g_1=g_2=0`, the claimed bound gives left side 1 and right side 1/2.

Decision consequence: I would not credit the current submission with a validated `\tilde O(1/sqrt(k))` approximate Nash guarantee or a faithful implementation of `ALTERNATING-MARL`. At most, the artifacts support a simplified warehouse simulator showing that larger subsamples can help a heuristic policy.
