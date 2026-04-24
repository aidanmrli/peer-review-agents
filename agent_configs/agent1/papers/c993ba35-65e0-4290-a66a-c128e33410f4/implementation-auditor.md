# Implementation Auditor Report

## Paper and Role

- Paper ID: `c993ba35-65e0-4290-a66a-c128e33410f4`
- Title: "Learning Approximate Nash Equilibria in Cooperative Multi-Agent Reinforcement Learning via Mean-Field Subsampling"
- Assigned role: Implementation Auditor
- Repository audited: `papers/c993ba35-65e0-4290-a66a-c128e33410f4/repos/alternating-marl`
- Repository HEAD: `2b1a57e151ac57d77fa8bbadbbd1de80a692320e` (`2b1a57e added TRPO`)
- Remote: `https://github.com/emiletimothy/alternating-marl`
- Permitted sources used: local paper PDF/source/artifacts, local repository clone, live Koala platform guide. I did not use OpenReview, citation counts, acceptance signals, or post-publication commentary.

## Task Scope

I inspected whether the released repository implements the paper's load-bearing methods and experiments:

- `G-LEARN` global-agent subsampled/mean-field Bellman learning.
- `L-LEARN` local-agent induced/chained MDP learning with a PAC episodic RL solver.
- `ALTERNATING-MARL` alternating best-response dynamics and `UPDATE` accept/reject/terminate rule.
- Online subsampling execution and evaluation.
- Figure/table reproduction scripts, dependencies, seeds, and hyperparameters.
- Consistency between the theoretical setup, numerical section, and code.

## Artifact Inventory

Local artifacts include:

- Paper source and PDF: `artifacts/main.tex`, `artifacts/sections/preliminaries.tex`, `artifacts/paper.pdf`, and `artifacts/source.tar.gz`.
- Paper figures used by LaTeX: `artifacts/sections/Figures/k_vs_n_rewards.png`, `k_vs_n_runtimes.png`, `robot2.png`, `zone_heatmap.png`.
- Repository clone and duplicated artifact repo: `repos/alternating-marl` and `artifacts/repo`.
- Repository code paths:
  - `scripts/alternating_marl.py`
  - `scripts/global_agent_optimizer.py`
  - `scripts/local_agent_optimizer.py`
  - `scripts/marl_example.py`
  - `scripts/plot_simulation.py`
  - `scripts/function_approximation/{agents.py,continuous_env.py,run_experiment.py,training.py}`
  - `scripts/hyperparameters.json`
  - `requirements.txt`
- Generated PNGs in the repo root include reward/runtime/mode/heatmap figures for tabular and continuous variants.

Missing or weak artifacts:

- No tests.
- No pinned environment, lockfile, `pyproject.toml`, `setup.py`, or `environment.yml`.
- No raw result tables, CSV logs, command transcripts, or saved random seeds supporting the included PNGs.
- No code implementing the paper's UCFH solver, explicit `k`-chained MDP, `|S_l|`-chained mean-field MDP, off-policy `Q`-learning extension, stochastic-reward extension, or federated optimization validation.

## Commands and Checks Run

- Fetched the live Koala guide: `curl -fsSL https://koala.science/skill.md`.
- Loaded local role instructions: `skills/implementation-auditor.md` and `skills/review-documentation-workflow.md`.
- Inventoried artifacts and repository files with `find`.
- Confirmed repository state with `git status --short`, `git rev-parse HEAD`, `git log --oneline -1`, and `git remote -v`.
- Searched paper/code claims with `rg -n "G-LEARN|L-LEARN|ALTERNATING|subsample|runtime|reward|seed|PPO|TRPO|A2C"`.
- Read paper algorithms and experiment section from:
  - `artifacts/sections/preliminaries.tex:147-324`
  - `artifacts/main.tex:1535-1620`
- Read code and configs with `sed`/`nl` for all main scripts.
- Syntax check: `python -m compileall -q scripts` passed. I removed the generated `__pycache__` directories afterward.
- Execution smoke test using the active Python failed before running experiments: `python -c "from scripts.marl_example import run_single_k; ..."` raised `ModuleNotFoundError: No module named 'numpy'`.
- The bundled untracked virtual environment in the local clone has `numpy 2.4.4` but failed on `matplotlib` import, so it is not a complete runnable environment for the plotting scripts.

## Paper Claims Checked

The paper states:

- `G-LEARN` should use empirical Bellman updates with `m` samples over either standard `S_l^k` states or mean-field count/distribution states (`artifacts/sections/preliminaries.tex:180-233`).
- `L-LEARN` should construct a `k`-chained or `|S_l|`-chained induced MDP and run an episodic PAC RL solver such as UCFH (`preliminaries.tex:236-264`).
- `ALTERNATING-MARL` should update global and local policies separately using an `UPDATE` routine that accepts, rejects, or terminates based on all-state value intervals with tolerance `2 eta` (`preliminaries.tex:266-306`).
- Online execution should sample `Delta` uniformly each timestep, query the learned global policy on sampled local states, execute all local policies, and collect the full-system average reward (`preliminaries.tex:308-324`).
- Experiments should use `n=1000`, `gamma=0.95`, `k={1,...,50}`, `m=30`, `N_steps=10`, 15 seeds per `k`, 50 rollouts per seed, and horizon 100 (`artifacts/main.tex:1583-1598`).
- The numerical section says the global reward is `r_g(s_g,a_g)=4-|s_g-a_g|` (`main.tex:1563-1567`) and claims experiments validate the theory at scale (`main.tex:1535-1536`, `1605-1618`).

## Paper-Code Matches

- The repository does contain a tabular alternating training driver (`scripts/alternating_marl.py`) and separate global/local optimizer files.
- The tabular environment has the same finite state/action sizes, population size, and discount as the paper config: 5 global states, 5 local states, 5 global actions, 3 local actions, `n_agents=1000`, `gamma=0.95` (`scripts/hyperparameters.json:2-8`).
- The local transition logits in `scripts/marl_example.py:53-74` match the paper's stay/move/drift/floor structure in `main.tex:1561-1562`.
- The local reward in `scripts/marl_example.py:90-113` matches the paper's aligned-state base reward and action bonus structure in `main.tex:1567-1571`.
- Evaluation in `scripts/alternating_marl.py:174-261` does implement per-step subsampling without replacement, global action lookup on count vectors, local stochastic actions, and full-population averaged reward.
- The repository includes scripts that generate reward/runtime and heatmap-style plots (`scripts/marl_example.py:175-212`, `scripts/plot_simulation.py:150-251`).

## Major Discrepancies

### 1. `G-LEARN` is not the empirical Bellman algorithm described in the paper

Paper: `G-LEARN` uses an empirical adapted Bellman operator with `m` random samples per update (`preliminaries.tex:219-232`), and switches between standard tuple states and mean-field parameterization depending on `|S_l|^k <= |S_l| k^{|S_l|}` (`preliminaries.tex:180-199`).

Code: `GlobalAgentOptimizer` always enumerates count vectors (`scripts/global_agent_optimizer.py:22-34`, `108-118`) and then uses a deterministic expected next-count transition rounded to an integer count vector (`global_agent_optimizer.py:127-176`). The Bellman backup is deterministic (`global_agent_optimizer.py:179-195`). The `n_mc` parameter is accepted and stored (`global_agent_optimizer.py:57-58`, `76-85`) but is not used in the transition computation or backup.

Impact: the central empirical Bellman-noise/sample-size parameter `m=30` is not actually load-bearing in the tabular implementation. The code implements a rounded mean-field value-iteration heuristic, not the algorithm analyzed in the paper.

Severity: critical for implementation fidelity.

### 2. `L-LEARN` omits the chained MDP and UCFH solver

Paper: `L-LEARN` should build a `k`-chained MDP or `|S_l|`-chained mean-field MDP and run UCFH with `epsilon_l=1/sqrt(k)` (`preliminaries.tex:236-264`).

Code: `LocalAgentOptimizer` collapses the local problem to a `(s_g, s_l)` value iteration using an effective global transition estimated from `Multinomial(k, mu)` samples (`scripts/local_agent_optimizer.py:22-29`, `85-116`). There is no chained state containing replicas, no micro-step serialization, no UCFH, no failure probability, no horizon `H`, and no `epsilon_l=1/sqrt(k)` parameter.

Additional mismatch: the paper says the representative local reward is scaled by `1/n` in the induced MDP (`preliminaries.tex:236-238`), while the code uses the raw local reward `r_l` in `R_arr` (`local_agent_optimizer.py:75-83`).

Impact: the local best-response guarantee in the paper is not reproduced by the code. The implementation is a heuristic local optimizer.

Severity: critical for implementation fidelity and reproducibility of the claimed approximate Nash guarantee.

### 3. `ALTERNATING-MARL` does not implement the paper's `UPDATE` rule or Nash certificate

Paper: after each proposed global or local update, `UPDATE` compares `V_new` and `V_old` with an explicit `2 eta` tolerance on all sampled states, then accepts, rejects, or terminates with a Nash certificate (`preliminaries.tex:281-303`).

Code: `train()` runs `g_learn()` and `l_learn()` back-to-back, then estimates one scalar Monte Carlo value using 20 rollouts of horizon 30 (`scripts/alternating_marl.py:281-292`). It stores the best scalar value, reverts only if the scalar decreases by a relative threshold, and terminates when adjacent scalar estimates are close (`alternating_marl.py:299-319`). There is no `eta`, no all-state interval comparison, no separate acceptance after each proposed global/local policy, and no certified approximate Nash stopping condition.

Impact: the training loop cannot substantiate the paper's convergence-to-`2 eta` approximate NE claim. Its early stopping is a heuristic performance plateau test, not the theoretical rule.

Severity: critical for the main algorithmic claim.

### 4. Evaluation transitions use the wrong global state for local transitions

Paper dynamics use local transitions conditioned on the current global state: `s_i(t+1) ~ P_l(. | s_i(t), s_g(t), a_i(t))` (`artifacts/sections/preliminaries.tex:80-82` and `main.tex:1561-1562`).

Code in evaluation first updates `s_g`, then uses the updated `s_g` to transition local agents (`scripts/alternating_marl.py:250-257`). This changes the system dynamics relative to the paper and relative to the model used in `GlobalAgentOptimizer`, where local count transitions use the current `s_g` (`global_agent_optimizer.py:169-176`).

The same pattern appears in `scripts/plot_simulation.py:101-114` and the function-approximation training path (`scripts/function_approximation/training.py:115-117`, `176-180`).

Impact: even if the policies trained by the code were accepted as approximations, the rollout metrics are generated under dynamics different from the formal model.

Severity: major.

### 5. Numerical hyperparameters and rewards do not match the paper

- Paper says `N_steps=10`; config uses `n_outer_iterations=5` (`main.tex:1593` vs `scripts/hyperparameters.json:21-23`).
- Paper says horizon 100; config uses evaluation horizon 50 and training-convergence horizon 30 (`main.tex:1546`, `1598`; `hyperparameters.json:25-27`; `alternating_marl.py:290-291`).
- Paper says global reward `r_g(s_g,a_g)=4-|s_g-a_g|`; code sets `r_g` to a flat constant 1.0 (`main.tex:1566`; `scripts/marl_example.py:77-87`). The README also states the global reward is flat, so the repo and paper disagree.
- Paper figures are named `k_vs_n_rewards.png` and `k_vs_n_runtimes.png` (`main.tex:1608-1611`), while the main script writes `reward_vs_k.png` and `runtime_vs_k.png` (`scripts/marl_example.py:194-209`). The paper figure files exist in the artifact source tree but are not generated under those names by the provided main script.
- Paper heatmap caption uses `k = 1, 10, 20, 35` (`main.tex:1616-1618`), while `plot_simulation.py` default supports those values but its `__main__` invocation uses only `[1, 10, 35]` (`scripts/plot_simulation.py:150-164`, `263-267`).

Impact: the released scripts do not directly reproduce the table configuration or paper figures as written.

Severity: major.

### 6. Function-approximation code is not the paper algorithm

The `function_approximation` directory is extra code, but it should not be counted as implementing the paper's theoretical algorithms. In `training.py`, the global "G-LEARN" step is supervised cross-entropy on `(histogram_k, true_mode)` labels (`scripts/function_approximation/training.py:53-84`, `135-137`), not subsampled mean-field Q-learning. The local step is REINFORCE/A2C/PPO/TRPO-style trajectory optimization (`training.py:86-141`), not the induced chained MDP plus UCFH procedure.

Impact: these scripts may produce additional PNGs but do not validate the paper's G-LEARN/L-LEARN theory.

Severity: moderate to major, depending on whether these figures are used as evidence.

### 7. Dependencies and seeds are insufficient for independent reproduction

- `requirements.txt` contains only unpinned package names: `numpy`, `matplotlib`, `seaborn`, `torch` (`requirements.txt:1-4`).
- There is no Python version lock or package hash.
- The active environment failed to import `numpy`; the bundled untracked local virtual environment has `numpy` but not `matplotlib`.
- Tabular evaluation seeds are exposed, but raw per-seed outputs are not saved. Figure PNG provenance is therefore not auditable.
- Function-approximation code uses PyTorch and `np.random` without a full deterministic seed discipline (`scripts/function_approximation/training.py:124-144`; agent action sampling uses global NumPy RNG in `agents.py`).

Impact: a reviewer cannot rerun the reported experiments in a clean environment with high confidence that they are reproducing the same results.

Severity: major for reproducibility.

## Reproducibility Blockers

1. No runnable pinned environment. The repository requirements are not versioned, and the local execution environment did not satisfy them.
2. No tests or validation scripts proving that the algorithms match the paper's definitions.
3. No raw numerical outputs backing the paper's plotted curves.
4. No direct script generating the paper's named `k_vs_n_rewards.png` and `k_vs_n_runtimes.png` from scratch.
5. No implementation of UCFH or the induced chained MDPs, so the local best-response claim is not independently checkable from code.
6. No implementation of the formal `UPDATE` rule, so the approximate Nash certificate is not independently checkable from code.
7. Evaluation rollout dynamics differ from the paper's stated Markov game.

## Severity and Decision Impact

Overall severity: critical.

The repository provides an illustrative simulator with some matching ingredients: count-vector global policies, random online subsampling, a tabular warehouse-like environment, and generated plots. It does not implement the paper's central algorithmic stack as specified. The largest gaps are the missing empirical Bellman sampling in `G-LEARN`, missing chained-MDP/UCFH construction in `L-LEARN`, missing `UPDATE` rule and Nash certificate in `ALTERNATING-MARL`, and a dynamics mismatch in evaluation.

Decision impact: the code should not be treated as reproducible evidence for the main claims that `ALTERNATING-MARL` learns a `tilde O(1/sqrt(k))` approximate Nash equilibrium or that the reported experiments validate the theory. At most, it supports a weaker claim that a simplified heuristic with count-vector subsampling can be simulated in a toy warehouse environment. For an ICML review, this is a substantial negative reproducibility finding and should materially lower confidence in the empirical and implementation-supported parts of the submission.

## Confidence

High for code/paper mismatch findings. The discrepancies are visible directly in the paper source and repository code. I could not run the full experiments because the current environment lacks the required installed packages and the bundled local virtual environment is incomplete, but the major algorithmic omissions do not depend on execution.
