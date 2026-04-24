# Correctness Specialist Report

Paper ID: `c993ba35-65e0-4290-a66a-c128e33410f4`

Title: "Learning Approximate Nash Equilibria in Cooperative Multi-Agent Reinforcement Learning via Mean-Field Subsampling"

Assigned role: Correctness Specialist

Task scope: Check definitions, theorem statements/proof obligations, sample-complexity claims, equilibrium approximation claims, experimental logic, metrics, and code-level assumptions for technical errors. This report uses only the paper source, local artifacts, and the provided repository clone. No OpenReview reviews, later decisions, citation data, or post-publication signals were used.

## Evidence Examined

- Paper source: `papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/main.tex`
- Main body source: `papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/sections/preliminaries.tex`
- PDF artifact: `papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/paper.pdf`
- Repository clone: `papers/c993ba35-65e0-4290-a66a-c128e33410f4/repos/alternating-marl`
- Key code inspected: `scripts/alternating_marl.py`, `scripts/global_agent_optimizer.py`, `scripts/local_agent_optimizer.py`, `scripts/marl_example.py`, `scripts/plot_simulation.py`, `scripts/hyperparameters.json`

Commands run:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,220p' skills/correctness-specialist.md
sed -n '1,220p' skills/review-documentation-workflow.md
rg -n "(Theorem|Lemma|Nash|mean-field|subsampl|sample|complexity|runtime|reward|Experiment)" papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/main.tex
nl -ba papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/sections/preliminaries.tex | sed -n '100,520p'
nl -ba papers/c993ba35-65e0-4290-a66a-c128e33410f4/artifacts/main.tex | sed -n '205,1635p'
nl -ba papers/c993ba35-65e0-4290-a66a-c128e33410f4/repos/alternating-marl/scripts/*.py
python3 -m py_compile scripts/alternating_marl.py scripts/global_agent_optimizer.py scripts/local_agent_optimizer.py scripts/marl_example.py scripts/plot_simulation.py
```

The code files passed `py_compile`; this only checks Python syntax, not correctness of the implemented method.

## Findings

### 1. Fatal: The Markov potential game proof is false for the stated model

Candidate error: The paper claims the induced Markov game is an exact Markov potential game with potential equal to the system reward, then uses that claim to prove convergence of alternating best-response dynamics.

Exact location:

- Potential-game definition: `artifacts/sections/preliminaries.tex:380-387`
- Claim and proof: `artifacts/main.tex:1103-1116`
- Main theorem relying on this structure: `artifacts/sections/preliminaries.tex:389-398`

Why it is technically wrong or unsupported:

The proof asserts that a unilateral local-agent deviation changes the local agent's utility by exactly the same amount as it changes the full system potential. This is not true in the stated model. The proposed potential is

```tex
\Phi(s,a) = r_g(s_g,a_g) + (1/n) \sum_i r_l(s_i,s_g,a_i)
```

but a single local agent's unilateral policy change can alter its own future state and, when sampled by the global policy, can alter the global action, the global transition, the global reward, and other local agents' future rewards through the global state. The potential difference therefore contains terms not present in the deviating local agent's reward. Even if the global action were independent of local state, the full potential change would equal only the deviating local term under additional assumptions; those assumptions are not stated or proved.

The equality in `main.tex:1108-1110` compares a local-agent reward difference to the full potential difference including `r_g` and all local rewards. This is the core step needed for exact potential structure, and it does not follow from additive rewards.

Evidence/derivation:

Consider a one-local-agent instance where the local reward is identically zero, but the global policy conditions on the local state and chooses a global action with nonzero `r_g`. A local policy deviation that changes the local state distribution can change the global action distribution and thus `r_g`. The local reward difference is zero, but the proposed potential changes. Hence the exact-potential equality fails.

Severity: fatal.

Acceptance consequence: The main convergence argument for `ALTERNATING-MARL` is not established. Without a valid potential function or a replacement convergence proof, the central approximate Nash equilibrium guarantee is unsupported.

### 2. Fatal: The claimed Nash equilibrium is stronger than the policy class actually optimized

Candidate error: The paper defines full `(n+1)`-player Nash and approximate Nash equilibria over arbitrary unilateral deviations, but the algorithm optimizes only a restricted global policy and one shared representative local policy.

Exact location:

- Restricted equilibrium discussion: `artifacts/sections/preliminaries.tex:98-102`
- Full Nash definitions: `artifacts/sections/preliminaries.tex:115-120`
- Claim that online execution yields approximate NE: `artifacts/sections/preliminaries.tex:308-323`
- Fixed point of sampled two-policy operator called a Nash equilibrium: `artifacts/main.tex:1161-1164`

Why it is technically wrong or unsupported:

The definitions at `preliminaries.tex:115-120` quantify over all global and individual local policies. The algorithm and analysis use `Pi_g^{(k)} x Pi_l`, where `Pi_l` is a shared homogeneous local policy class and `Pi_g^{(k)}` observes only `k` local states or a histogram. A fixed point in this restricted two-policy class is at most a symmetric restricted equilibrium, not a full Nash equilibrium against arbitrary individual local deviations or unrestricted global deviations.

The paper does acknowledge restricted policy classes at `preliminaries.tex:98-102`, but the formal definitions and theorems switch back to full Nash language without proving equivalence. Homogeneity does not by itself justify replacing every individual local deviation by a representative shared-policy deviation.

Evidence/derivation:

For deterministic policies, the full local deviation set allows agent `i` to change only its own policy while all other local agents keep the shared policy. `L-LEARN` instead learns one common policy to deploy to all local agents. These are different optimization problems. In particular, a unilateral deviating local agent may exploit its effect on the sampled global observation differently from a policy applied symmetrically to every local agent.

Severity: fatal.

Acceptance consequence: The main theorem's "approximate Nash Equilibrium" statement is overclaimed. The result would need to be restated as a restricted symmetric equilibrium and then reproved for that solution concept.

### 3. Fatal: Lemma `expectation_expectation_max_swap` is mathematically false

Candidate error: The paper uses a false inequality to move a maximum outside expectations in the Lipschitz proof for the subsampled Q-functions.

Exact location:

- Lemma statement: `artifacts/main.tex:485-488`
- Proof: `artifacts/main.tex:489-503`
- Used in the Lipschitz induction: `artifacts/main.tex:367-372` and `artifacts/main.tex:418-425`

Why it is technically wrong or unsupported:

The lemma states, in effect,

```tex
| E[max_a f_a(X)] - E[max_a g_a(X)] |
<= max_a | E[f_a(X) - g_a(X)] |.
```

This inequality is false. The correct generic bound is

```tex
| E[max_a f_a(X)] - E[max_a g_a(X)] |
<= E[max_a |f_a(X)-g_a(X)|],
```

which cannot in general be replaced by `max_a |E[...]|`.

Evidence/derivation:

Let `X` be uniform on `{0,1}`. Let `f_1(X)=1{X=0}`, `f_2(X)=1{X=1}`, and `g_1(X)=g_2(X)=0`. Then

```text
left side = |E[max(f_1,f_2)] - E[0]| = 1
right side = max(E[f_1], E[f_2]) = 1/2
```

so the lemma fails. This is not a constant-factor issue; it invalidates the inductive step used to prove the central Q-function Lipschitz bound.

Severity: fatal.

Acceptance consequence: The main `O(1/sqrt(k))` global best-response approximation bound rests on this Lipschitz proof. As written, the proof is invalid.

### 4. Major: The DKW-to-TV conversion is false and removes a missing dimension factor

Candidate error: The proof converts a sup-norm empirical-distribution bound into a total-variation bound using an invalid equivalence.

Exact location:

- DKW-style sampling theorem: `artifacts/main.tex:553-556`
- TV conversion and bound: `artifacts/main.tex:560-575`

Why it is technically wrong or unsupported:

At `main.tex:566`, the paper states

```tex
TV(F_delta, F_[n]) <= epsilon iff sup_x |F_delta - F_[n]| < 2 epsilon.
```

This is false for multi-category finite spaces. Since

```tex
TV(p,q) = (1/2) sum_x |p(x)-q(x)|,
```

a sup-norm bound `sup_x |p(x)-q(x)| <= alpha` implies only

```tex
TV(p,q) <= (|S_l|/2) alpha.
```

Conversely, `TV(p,q) <= epsilon` implies `sup_x |p(x)-q(x)| <= 2 epsilon`, but not the reverse without a dimension factor.

Evidence/derivation:

For `|S_l|=3`, take `p=(1,0,0)` and `q=(1-2e,e,e)`. Then `sup_x |p-q| = 2e`, but `TV(p,q)=2e`, not `e`. For larger state spaces the gap scales with `|S_l|`.

Severity: major.

Acceptance consequence: The advertised value-error and sample-complexity bounds hide a missing dependence on the local state-space size. More importantly, the stated theorem is not proved as written.

### 5. Major: The policy-counting and finite-step convergence bounds use the number of state-action pairs, not the number of policies

Candidate error: The paper bounds the number of best-response iterations by `2 |Pi_g| |Pi_l|`, but then evaluates `|Pi_g|` and `|Pi_l|` as if they were state-action table sizes rather than policy-class cardinalities.

Exact location:

- Earlier corollary: `artifacts/main.tex:1146-1149`
- Approximate convergence bound: `artifacts/main.tex:1305-1337`
- Main theorem's chosen `N_steps`: `artifacts/sections/preliminaries.tex:391-398`

Why it is technically wrong or unsupported:

For a finite deterministic policy class mapping a state set `X` to an action set `A`, the number of policies is `|A|^{|X|}`, not `|X| |A|`. For the mean-field global policy class, the deterministic policy count is at least

```text
|A_g|^( |S_g| * |mu_k(S_l)| )
```

where `|mu_k(S_l)| = binom(k+|S_l|-1, |S_l|-1)`. For the local policy class it is

```text
|A_l|^( |S_g| * |S_l| ).
```

If stochastic policies are allowed, the policy class is infinite unless discretized. The paper instead uses products such as

```tex
|S_g|^2 |S_l|^2 k^{|S_l|} |A_g| |A_l|
```

at `main.tex:1324-1335`. That is a count of table entries or a loose state-action-space size, not a count of policies.

Severity: major.

Acceptance consequence: The finite-iteration convergence and total sample-complexity theorem are not valid. The advertised polynomial dependence follows from an incorrect cardinality calculation.

### 6. Major: The interval-stopping rule does not certify a `2 eta` approximate Nash equilibrium

Candidate error: The `UPDATE` function stops when two estimated values are within `2 eta` and claims this certifies a `2 eta`-NE.

Exact location:

- Algorithmic rule: `artifacts/sections/preliminaries.tex:296-304`
- Proof of interval correctness: `artifacts/main.tex:1186-1303`

Why it is technically wrong or unsupported:

The proof compares estimated joint values of consecutive policies, not the unilateral best-response gaps required by an approximate Nash condition. A small difference between `V(pi_g', pi_l')` and `V(pi_g, pi_l)` does not imply that no unilateral deviation from `(pi_g, pi_l)` can improve by more than `2 eta`. Sequential updates can mask a profitable deviation by changing both policies, and estimated rollout/subsampled values are not shown to uniformly bound all unilateral deviations over the policy class.

There is also an internal inconsistency in the convergence proof. The lemma fixes `epsilon > 0` at `main.tex:1305`, but then uses `Delta_i(pi) > eta` at `main.tex:1308` and concludes `Delta_i(pi)-eta > epsilon-eta > 0` at `main.tex:1312`, which only follows if `Delta_i(pi) > epsilon`.

Severity: major.

Acceptance consequence: The algorithm's termination certificate is not technically justified. This directly affects whether the returned policy can be called an approximate Nash equilibrium.

### 7. Major: The local-agent best-response reduction does not prove a stationary shared local best response

Candidate error: The paper reduces `L-LEARN` to a `k`-chained or mean-field chained MDP and invokes UCFH, but the constructed MDP and solver do not establish the claimed stationary shared-policy best response.

Exact location:

- `L-LEARN` description: `artifacts/sections/preliminaries.tex:236-264`
- `k`-chained MDP: `artifacts/main.tex:940-989`
- mean-field chained MDP: `artifacts/main.tex:991-1046`
- local best-response theorem: `artifacts/main.tex:1048-1074`

Why it is technically wrong or unsupported:

In the `k`-chained MDP, the reward is placed only at the `j=1` micro-node:

```tex
1{j=1} gamma^{t(tau)} (1/n) r_l(s_1, s_g, a)
```

at `main.tex:966`. The actions at `j=2,...,k` affect future replica states and possibly future global actions but receive zero immediate local reward. UCFH on this expanded nonstationary MDP returns a policy over expanded micro-states, not necessarily a stationary shared policy of the form `pi_l(s_l,s_g)` applied identically to every local agent. The proof does not give a projection from the UCFH policy back to the claimed local policy class with preserved value.

The mean-field version similarly uses aggregate histogram transitions and a tagged local state, but the proof only counts state-space sizes and invokes UCFH; it does not prove that the resulting MDP's optimal policy is equivalent to a representative local best response in the original sampled Markov game.

Severity: major.

Acceptance consequence: The `epsilon_l` best-response assumption used by the main theorem is not established.

### 8. Major: The implementation does not implement the theoretical algorithms used in the proofs

Candidate error: The released code diverges materially from the algorithms and assumptions in the paper.

Exact location:

- Paper `G-LEARN` uses Monte Carlo empirical Bellman updates: `artifacts/sections/preliminaries.tex:180-233`
- Code `GlobalAgentOptimizer` ignores `n_mc` and uses deterministic rounded expected transitions: `repos/alternating-marl/scripts/global_agent_optimizer.py:76-85`, `127-188`
- Paper `L-LEARN` invokes chained MDP plus UCFH: `artifacts/sections/preliminaries.tex:241-257`
- Code `LocalAgentOptimizer` uses value iteration on `(s_g,s_l)` with an MC effective global transition from a stationary mean-field distribution: `repos/alternating-marl/scripts/local_agent_optimizer.py:85-116`
- Training loop uses rollout-based best-value reversion rather than the paper's all-state `2 eta` interval rule: `repos/alternating-marl/scripts/alternating_marl.py:266-320`

Why it is technically wrong or unsupported:

The global optimizer accepts an `n_mc` parameter but never samples next count vectors in the Bellman backup. Instead, it computes an expected next count vector and rounds it to a valid integer count vector (`global_agent_optimizer.py:160-177`). This is a biased deterministic approximation, not the empirical Bellman operator analyzed in the paper.

The local optimizer does not construct either chained MDP and does not run UCFH. It samples count vectors from a stationary distribution `mu` independent of the tagged local agent's state/action (`local_agent_optimizer.py:87-95`), then solves a reduced MDP. This omits the non-Markovian dependence that the paper identifies as the central difficulty.

Severity: major.

Acceptance consequence: The numerical evidence cannot validate the theoretical algorithm as stated. It validates a different heuristic.

### 9. Moderate to major: The experimental configuration and environment described in the paper disagree with the released code

Candidate error: The paper reports experiment settings and reward definitions that are inconsistent with the repository.

Exact location:

- Paper reward definition: `artifacts/main.tex:1563-1571`
- Code global reward: `repos/alternating-marl/scripts/marl_example.py:77-87`
- Paper hyperparameter table: `artifacts/main.tex:1578-1602`
- Code hyperparameters: `repos/alternating-marl/scripts/hyperparameters.json:21-28`
- Code heatmap main entry: `repos/alternating-marl/scripts/plot_simulation.py:262-268`

Why it is technically wrong or unsupported:

The paper states `r_g(s_g,a_g)=4-|s_g-a_g|` at `main.tex:1566`, but the code sets every global reward to `1.0` at `marl_example.py:77-87`. The table states `N_steps=10` and evaluation horizon `100` at `main.tex:1593-1598`, while the code config uses `n_outer_iterations=5` and `horizon=50` at `hyperparameters.json:21-28`. The plotting script's main function generates a heatmap only for `[1, 10, 35]` (`plot_simulation.py:262-268`), while the caption reports panels for `k=1,10,20,35` (`main.tex:1617`).

Severity: moderate to major.

Acceptance consequence: The empirical section is not a reliable description of the released experiment. Claims about runtime, horizon, and the specific reward model cannot be independently checked from the code as written.

### 10. Major: The stochastic-reward extension applies Hoeffding with the wrong scaling

Candidate error: The stochastic-reward proof derives a concentration bound for averaging `Xi` reward samples, but the algebra loses a factor of `sqrt(Xi)`.

Exact location:

- Stochastic reward extension: `artifacts/main.tex:1461-1531`
- Hoeffding step and rearrangement: `artifacts/main.tex:1519-1525`

Why it is technically wrong or unsupported:

If `rho` is the sum of `Xi` independent bounded samples with range `B`, then

```text
Pr(|rho/Xi - E R| >= a) <= 2 exp(-2 Xi a^2 / B^2).
```

Solving gives `a = B sqrt(log(2/delta)/(2 Xi))`. The proof instead writes a bound with `Xi^2` in the denominator after rearrangement (`main.tex:1523`), leading to a much smaller error and the choice `Xi = O(k^{1/4} sqrt(log k))` at `main.tex:1524`. To achieve an `O(1/sqrt(k))` reward-sampling error using Hoeffding, the required `Xi` should scale on the order of `k log k` up to range factors, not `k^{1/4}`.

The proof also averages the randomized Bellman operator, which contains future value terms, but compares `rho/Xi` only to `E[R_delta(s,a)]`.

Severity: major for the extension, moderate for the main deterministic claim.

Acceptance consequence: The stochastic-reward generalization is not proved.

### 11. Moderate: The empirical adapted Bellman operator is not well-defined as written

Candidate error: The empirical adapted Bellman operator has inconsistent arguments and undefined action notation.

Exact location:

- Definition: `artifacts/main.tex:221-224`

Why it is technically wrong or unsupported:

The operator is defined on `F_{s_{Delta \setminus j}}` but the right side uses `r_Delta`, samples `s_Delta^ell`, and evaluates `a_g^ell`, which is not defined. The global-action maximization should evaluate `a_g'`, not a sampled `a_g^ell`, unless the algorithm is sampling actions from a behavior policy; no such behavior policy is introduced for this offline operator.

Severity: moderate.

Acceptance consequence: This may be partly notational, but it obscures the exact operator whose fixed point and sample complexity are analyzed.

## Limitations or Blockers

- I did not rerun the full experiments because the role task was correctness audit rather than reproduction, and the code/paper mismatches are already directly visible from source.
- I did not inspect external literature beyond permitted local references and the live platform guide. Literature novelty is covered by the Literature Specialist role, not this report.
- The code syntax check passed, but no unit tests or raw experiment logs are provided in the repository.

## Confidence Level

High. The most serious findings rely on direct algebraic contradictions or explicit source-code mismatches:

- The max/expectation lemma is false by counterexample.
- The TV equivalence is false for finite spaces with more than two states.
- The potential-game equality omits global reward and other-agent terms affected by a local deviation.
- The policy-counting bound confuses table size with policy-class cardinality.
- The repository implements a different algorithm than the paper analyzes.

## Decision Impact

The correctness issues materially undermine the acceptance case. The central theorem that `ALTERNATING-MARL` learns a `tilde O(1/sqrt(k))` approximate Nash equilibrium is not proved as written, and the code does not implement the proof-level algorithm. My correctness recommendation is a substantial downgrade: the paper may contain an interesting modeling direction, but the current theoretical and empirical evidence is not technically sound enough for acceptance without major revision.
