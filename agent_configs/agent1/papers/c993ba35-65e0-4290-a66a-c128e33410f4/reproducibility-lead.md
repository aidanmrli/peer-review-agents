# Reproducibility Lead Report

Paper ID: `c993ba35-65e0-4290-a66a-c128e33410f4`

Title: "Learning Approximate Nash Equilibria in Cooperative Multi-Agent Reinforcement Learning via Mean-Field Subsampling"

Assigned role: Reproducibility Lead

Task scope: Coordinate the internal review, identify the central claims, compare independent role findings, and decide how reproducibility affects the public comment and later verdict calibration.

## Central Claims Tested

1. `ALTERNATING-MARL` learns a `\tilde O(1/sqrt(k))` approximate Nash equilibrium with high probability under the stated cooperative global/local Markov game.
2. The global and local subroutines are reproducible from the released artifacts: `G-LEARN` as subsampled empirical Bellman learning, `L-LEARN` as a chained-MDP/UCFH best-response solver, and `UPDATE` as the stated accept/reject/termination certificate.
3. The numerical warehouse/robotics experiment can be reproduced from the linked repository and supports the stated k tradeoff.
4. The novelty claim is correctly grounded relative to prior mean-field MARL, major-minor mean-field MARL, Markov potential games, UCFH, and Nash Q-learning.

## Minimum Reproduction Target

Before seeing role outcomes, I set the minimum acceptable reproduction target as follows:

- Two independent internal roles should be able to reproduce either the central `O(1/sqrt(k))` claim at the level of proof/code trace or the reported k-dependent empirical trend under the stated experimental setup.
- The implementation auditor should verify that the linked GitHub repository actually implements the paper's `G-LEARN`, `L-LEARN`, and `ALTERNATING-MARL` procedures closely enough that execution evidence supports the paper rather than a different heuristic.
- The correctness specialist should find no fatal gap in the potential-game, max/expectation, or sample-complexity reasoning.

## Role Evidence Compared

### Independent Reproducer A

Reproducer A attempted the official repository path first. The default `python3 scripts/marl_example.py` failed in the base environment with `ModuleNotFoundError: No module named 'numpy'`. After creating `.venv` and installing `requirements.txt`, the full driver timed out after 180 seconds without producing buffered output. A reduced public-API run over `k=1,5,10` with two evaluation seeds produced increasing means:

```text
k=1 mean=83.1134 std=0.1055 train=0.723 wall=1.733
k=5 mean=85.7163 std=0.9774 train=0.771 wall=1.784
k=10 mean=87.2679 std=1.5376 train=0.458 wall=1.471
```

Outcome: partial reproduction of a weak qualitative trend, but not of the full reported experiment or central theorem.

### Independent Reproducer B

Reproducer B used a source/configuration route. It found direct mismatches between the paper and repository: paper global reward `4-|s_g-a_g|` versus code reward `1.0`, paper `N_steps=10` and horizon 100 versus code `n_outer_iterations=5` and horizon 50, heatmap/figure generation mismatch, and omission of empirical Bellman sampling, chained MDP/UCFH, and formal `UPDATE`.

Outcome: mismatch for central implementation reproducibility.

### Implementation Auditor

The implementation auditor concluded that the repository is an illustrative simulator, not a faithful implementation of the claimed algorithmic stack. Critical findings:

- `GlobalAgentOptimizer` stores `n_mc` but uses deterministic rounded expected transitions and deterministic Bellman backup.
- `LocalAgentOptimizer` solves a reduced `(s_g,s_l)` value iteration with an effective global transition, not a chained MDP/UCFH problem.
- `AlternatingMARL.train()` uses scalar rollout reversion/plateau stopping, not the paper's all-state `2 eta` interval `UPDATE`.
- Evaluation transitions use the updated global state for local transitions, whereas the paper dynamics condition local transitions on the current global state.
- Dependencies are unpinned and no raw logs/tests are provided.

Outcome: critical implementation-reproducibility failure.

### Correctness Specialist

The correctness specialist found several proof-level failures. The most decision-relevant:

- The Markov potential proof equates a unilateral local reward difference with a full-system potential difference, but a local policy change can affect global actions/rewards and other agents' future rewards through sampled global observations.
- The formal Nash definitions quantify over unrestricted global and individual-local deviations, while the algorithm optimizes a restricted global policy and one shared representative local policy. The proved object, if any, is a restricted symmetric equilibrium.
- The lemma moving `max` outside expectation is false by counterexample, undermining the subsampled Q-function Lipschitz proof.
- The DKW-to-TV conversion omits a local-state dimension factor.
- The convergence/sample-complexity proof counts table entries as if they were deterministic policy cardinalities.

Outcome: central theorem not established as written.

### Literature Specialist

The literature specialist found plausible but narrower novelty. The global-agent subsampling machinery substantially follows Anand et al. 2025; UCFH is external; major-minor mean-field MARL and classic Nash Q-learning are under-integrated; and the empirical section lacks relevant baselines. The novelty could still be meaningful if the chained-MDP and convergence proof were correct, but the correctness and implementation findings substantially reduce that credit.

Outcome: weak-to-moderate novelty/framing concern that becomes major after correctness failures.

## Reproduction Outcome Across Independent Reproducers

The central claim was not reproduced by two independent internal roles.

- Reproducer A recovered only a small qualitative k trend using a reduced version of a heuristic implementation.
- Reproducer B found that the implementation and paper setup diverge too much for the artifact to count as a reproduction of the paper's algorithm.
- The implementation auditor and correctness specialist independently corroborated that the proof-level and code-level algorithms do not align.

Reproducibility classification: weak reproducibility for the central claims; partial reproducibility only for an illustrative toy trend.

## Decision Impact

This is a substantial negative finding. The paper may contain an interesting modeling direction, but the main acceptance case depends on a theorem and implementation support that did not survive internal reproduction. I would materially mark down technical soundness and reproducibility. For later verdict calibration, this paper belongs in the reject range unless other agents provide a convincing repair of the potential-game/equilibrium proof and a faithful implementation path.

## Remaining Uncertainty

Some issues could be repaired by restating the guarantee as a restricted symmetric equilibrium and replacing the flawed proof steps. The current submission, however, does not provide those repairs. The public comment should therefore avoid claiming the idea is impossible; it should state that the reported theorem and artifacts do not currently establish the claimed result.

Confidence: high.
