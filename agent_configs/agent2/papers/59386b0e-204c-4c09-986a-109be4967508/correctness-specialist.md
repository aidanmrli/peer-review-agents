# Correctness Specialist Report

Paper: "Graph-GRPO: Training Graph Flow Models with Reinforcement Learning"
Koala paper ID: `59386b0e-204c-4c09-986a-109be4967508`
Role: Correctness Specialist
Date: 2026-04-24

## Scope and Evidence

I checked the method, derivation, GRPO objective, refinement description, V.U.N. computation, and molecular optimization metric claims using only the supplied artifacts:

- PDF/source: `papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/paper.pdf`, `main.tex`, `ref.bib`
- Code clone: `papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG`, commit `365bda9affadd5c2307014a0532ddaa244399441`

Commands and checks:

- `rg -n "transition|analytical|probab|GRPO|objective|refinement|VUN|valid|unique|novel|SA|QED|optimization|metric|reward" artifacts/main.tex`
- `rg -n "GRPO|PPO|advantage|clip|refinement|prescreen|oracle|PMO|dock|vina|reward|kl|policy" artifacts/DeFoG -g '!*.pdf'`
- `nl -ba artifacts/main.tex | sed -n '440,510p'`, `531,610p`, `674,712p`, `915,1124p`, `1260,1375p`, `1440,1475p`, `1537,1598p`
- `nl -ba artifacts/DeFoG/src/graph_discrete_flow_model.py | sed -n '638,664p'`
- `nl -ba artifacts/DeFoG/src/analysis/spectre_utils.py | sed -n '829,877p'`
- PMO sum recomputation from Table 3 values: prescreened sum = `19.270`, cold-start sum = `18.987`; these match the reported totals.
- Analytical counterexample for discrete transition normalization using Eq. (closed-form rate): with `S=4`, `p0=[0.01,0.70,0.19,0.10]`, current state `0`, `p_theta=[0,0.8,0.1,0.1]`, and final-step `dt/(1-t)=1`, off-diagonal Euler probabilities sum to `10.525`, so the "stay" probability is negative before clipping and the row remains unnormalized after diagonal clipping.

## Candidate Errors

### 1. The discrete-time analytical transition probability is not guaranteed to be a valid probability distribution

Location:

- Paper preliminary CTMC transition: `main.tex` lines 301-309.
- Analytical rate matrix: `main.tex` lines 492-500 and Appendix lines 1353-1364.
- Discrete implementation statement: `main.tex` lines 1368-1372.
- Code implementation: `DeFoG/src/graph_discrete_flow_model.py` lines 638-664.

Claim being tested:

The paper claims that off-diagonal transition probabilities can be computed as `p(z_{t+Delta t}|z_t) approx R_t^theta(z_t,z_{t+Delta t}) Delta t`, with the diagonal obtained by subtracting the off-diagonal mass from 1, and that this "ensures" normalization and numerical stability.

Why this is technically wrong or unsupported:

The CTMC rate in Eq. (closed-form rate) has a factor `1 / ((1-t) p0(z_t))`. When `p0(z_t)` is small, or when the step is near `t=1`, off-diagonal Euler probabilities can sum above 1. In that case the diagonal probability `1 - sum_offdiag` is negative. The paper does not provide a CFL-style step-size bound or a proof that the chosen schedule keeps the off-diagonal mass below 1 for non-uniform priors.

The released DeFoG implementation follows exactly the risky operation: it computes `step_probs_X = R_t_X * dt` and `step_probs_E = R_t_E * dt` at lines 638-640, zeros the diagonal at lines 645-646, then writes the diagonal as `(1.0 - sum).clamp(min=0.0)` at lines 649-657. If the off-diagonal mass is greater than 1, this clamp only prevents a negative diagonal; it does not renormalize the row. The returned tensor at lines 660-664 can therefore have row sum greater than 1.

Evidence or derivation:

For four categories with a full-support but non-uniform prior `p0=[0.01,0.70,0.19,0.10]`, current category `z_t=0`, and model prediction `p_theta=[0,0.8,0.1,0.1]`, Eq. (closed-form rate) gives off-diagonal final-step probabilities `[6.20, 2.05, 2.275]` when `dt/(1-t)=1`. Their sum is `10.525`. The diagonal becomes negative before clipping; after the code's clamp it is 0 and the row sum remains `10.525`, not 1.

Severity: major.

Consequence for acceptance:

This directly undermines the central technical claim that the analytical transition probabilities make rollouts well-defined and differentiable for RL. Sampling with unnormalized weights may still run in PyTorch, but cached "probabilities", policy ratios, and KL terms are no longer the transition probabilities claimed in the method.

### 2. The provided GitHub clone does not contain the claimed Graph-GRPO implementation

Location:

- Paper states Graph-GRPO records transition probabilities and performs GRPO training in `main.tex` lines 536-549 and 583-603.
- Artifact clone search found no implementation for GRPO/refinement/PMO/docking policy optimization in `artifacts/DeFoG`; `rg -n "GRPO|PPO|advantage|clip|refinement|prescreen|oracle|PMO|dock|vina|reward|kl|policy" artifacts/DeFoG -g '!*.pdf'` returned only generic DeFoG/PyTorch Lightning references, README text, and unrelated occurrences.
- The code inspected at `DeFoG/src/flow_matching/rate_matrix.py` lines 25-73 still samples a pseudo clean graph at lines 32-38 before constructing the rate, i.e. the original Monte Carlo-style DeFoG path rather than the paper's analytical expectation.

Claim being tested:

The paper claims a new online RL framework with differentiable analytical transition probabilities, GRPO rollouts, KL-regularized policy updates, refinement, and PMO/docking optimization.

Why this is technically wrong or unsupported:

The supplied repository is a DeFoG clone, not an implementation of the paper's claimed method. It has no visible GRPO training loop, no advantage computation, no PPO clipping implementation, no refinement priority pool, no PMO oracle loop, and no docking reward implementation corresponding to the paper. The available rate-matrix code still samples `X_1_pred`/`E_1_pred` via `sample_discrete_features` in `rate_matrix.py` lines 32-38.

Severity: major.

Consequence for acceptance:

The algorithmic claims cannot be checked against code. This is a correctness limitation because several paper equations are implementation-sensitive: policy ratios require exact action log-probabilities, KL requires a full distribution or a justified estimator, and refinement requires exact oracle accounting. The artifact provided does not substantiate those claims.

### 3. The KL regularizer is not the KL divergence stated in the text

Location:

- Objective: `main.tex` lines 583-592.
- KL formula: `main.tex` lines 600-603.

Claim being tested:

The paper says it uses `D_KL(pi_theta || pi_ref)` to prevent the RL-optimized model from deviating from the base model.

Why this is technically wrong or unsupported:

The displayed expression

`pi_theta(G_{t+Delta t}|G_t) log [pi_theta(G_{t+Delta t}|G_t) / pi_ref(G_{t+Delta t}|G_t)]`

is a single-action term, not the categorical KL divergence unless it is summed or expected over all possible next graph states/actions. For a graph transition, the action space factorizes over many node and edge dimensions; a correct KL would need either a full sum over each factor's categorical support, a product/factorized derivation, or a sampled KL estimator with bias/variance justification. The paper provides none of these.

Severity: moderate to major.

Consequence for acceptance:

The regularization term is central to the claim that reward hacking and chemical invalidity are controlled. As written, the mathematical objective is incomplete and does not justify the claimed KL behavior.

### 4. The GRPO clipping equation conflicts with the stated asymmetric PPO configuration

Location:

- GRPO surrogate: `main.tex` lines 594-599.
- Training configuration: `main.tex` lines 1451-1454.

Claim being tested:

The paper claims standard GRPO training and later states it uses asymmetric PPO clipping with `epsilon_low=0.2` and `epsilon_high=0.28`.

Why this is technically wrong or unsupported:

The formal objective uses `clip(r, 1 +/- epsilon)` with a single symmetric epsilon at line 595. The appendix states asymmetric clipping at line 1453, but the objective never defines the asymmetric clipping interval `[1-epsilon_low, 1+epsilon_high]`. This is not just notation: asymmetric clipping changes the optimization objective and can materially affect policy updates.

Severity: moderate.

Consequence for acceptance:

This makes the training objective ambiguous. Reproducing the reported method from the paper alone would require guessing which objective was actually used.

### 5. The refinement configuration is internally inconsistent

Location:

- Refinement method: `main.tex` lines 693-708.
- Perturbation ablation: `main.tex` lines 1077-1089 and discussion lines 1177-1181.
- Appendix configuration: `main.tex` lines 1457-1468.

Claim being tested:

The paper presents refinement as a fixed method and states all reported results are averaged over seeds with a specified refinement setup.

Why this is technically wrong or unsupported:

The main ablation evaluates `t_epsilon` values `0.9`, `0.7`, `0.5`, `0.3`, and `0.0` and says best results are obtained at `0.7` or `0.9`. The implementation appendix then states that refinement is performed with a fixed noise level `t_epsilon=0.8` across all benchmarks. The table caption says the ablation is seed 0 only, while line 1457 says all molecular optimization results are averaged over seeds 0, 1, and 2. The exact `t_epsilon` used for Table 3 and the ablation conclusions is therefore ambiguous.

Severity: moderate.

Consequence for acceptance:

The refinement gain is a major empirical claim, especially for PMO. The paper does not specify a reproducible selection rule for the refinement noise level or clearly separate seed-0 tuning from final multi-seed evaluation.

### 6. Prescreened PMO results are described as state-of-the-art despite using an explicitly larger oracle budget

Location:

- PMO setup: `main.tex` lines 1099-1106.
- Results claim: `main.tex` lines 1117-1124.
- Table 3: `main.tex` lines 962-1000.

Claim being tested:

The paper claims the prescreened setting advances performance to a new state-of-the-art AUC-top10 score.

Why this is technically wrong or unsupported:

The PMO setup says the benchmark uses a strict budget of 10,000 oracle calls, but the prescreening setting first evaluates the entire ZINC250k dataset, consuming 250,000 oracle calls, before the optimization budget. The table separates prescreening and cold-start columns, which is good, but the prose at line 1122 calls the prescreened result a new state-of-the-art without restating that it uses an additional 250k calls. This is not an equal-budget PMO claim.

Evidence or derivation:

The reported AUC-top10 sums in Table 3 are internally consistent: summing the 23 Graph-GRPO prescreened values gives `19.270`, and summing the cold-start values gives `18.987`. The numerical table arithmetic is correct. The issue is the interpretation of the prescreened number as a state-of-the-art benchmark result under a nominal 10,000-call PMO protocol.

Severity: moderate.

Consequence for acceptance:

The prescreened result should not be treated as equal evidence against methods that did not spend the 250k prescreening calls, unless the comparison is explicitly framed as a separate high-budget variant.

### 7. V.U.N. computation itself is standard, but one synthetic-graph conclusion is overstated

Location:

- Synthetic setup and result discussion: `main.tex` lines 932-958.
- Synthetic table: `main.tex` lines 732-752.
- V.U.N. implementation: `DeFoG/src/analysis/spectre_utils.py` lines 829-877.

Claim being tested:

The paper claims Graph-GRPO outperforms graph diffusion models with 1,000 steps on the synthetic graph benchmarks.

Why this is technically wrong or unsupported:

The V.U.N. computation in the provided DeFoG code is recognizable: it counts generated graphs that are unique, non-isomorphic to training graphs, and valid, divided by all generated graphs (`spectre_utils.py` lines 829-877). The table values are plausible increments for 40 generated graphs per evaluation.

However, the broad "outperforms graph diffusion models" statement is not uniformly true across reported metrics. On Tree, DiGress has a better ratio (`1.6`) than Graph-GRPO (`2.2`) in Table 1, even though Graph-GRPO has better V.U.N. On Planar, Graph-GRPO ties DeFoG in V.U.N. (`95.0`) and improves ratio, so the improvement is metric-specific.

Severity: minor to moderate.

Consequence for acceptance:

This is not a fatal technical error, but the conclusion should be narrowed to "improves V.U.N. on Tree and ratio on Planar relative to DeFoG" rather than a blanket synthetic benchmark dominance claim.

## Final Synthesis and Score Impact

The central analytical CTMC derivation is plausible at the continuous-rate level, but the paper's conversion of rates into discrete transition probabilities is not technically justified and can produce invalid, unnormalized transition rows under non-uniform priors. This is decision-relevant because Graph-GRPO's policy ratio and KL terms depend on these quantities being actual transition probabilities.

The GRPO objective and KL term are under-specified, the refinement hyperparameter story is inconsistent, and the provided artifact is a DeFoG clone without the claimed GRPO/refinement/oracle implementation. Simple PMO table sums are correct, and the V.U.N. code path is broadly standard, but several interpretation claims are overstated.

Correctness impact: major negative. I would materially discount the paper's method claims unless the authors provide a corrected discrete transition treatment, the actual Graph-GRPO implementation, and a clarified/refactored RL objective and refinement protocol.
