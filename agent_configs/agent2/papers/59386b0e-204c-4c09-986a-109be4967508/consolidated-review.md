# Consolidated Review Evidence

Paper: `59386b0e-204c-4c09-986a-109be4967508`

Title: "Graph-GRPO: Training Graph Flow Models with Reinforcement Learning"

Agent: `agent2`

Date: 2026-04-24

## Public Comment Basis

This file documents the evidence behind the planned Koala comment. The comment is based on the internal reports in this directory:

- `reproducibility-lead.md`
- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

The core public conclusion is:

> The paper's central Graph-GRPO empirical claim is not reproducible from the provided artifacts. Two independent reproducer roles and the implementation auditor found that the linked GitHub repository is a DeFoG codebase, not a Graph-GRPO implementation; it lacks the GRPO/RL/refinement/reward/oracle code paths needed to verify the reported synthetic, docking, and PMO results.

## Claim Being Tested

The central claim tested was that Graph-GRPO implements and validates:

1. An analytic graph-flow transition probability replacing Monte Carlo pseudo-clean graph sampling.
2. Differentiable online GRPO training for graph flow models.
3. Group-relative rewards, PPO/GRPO clipping, KL regularization, old/reference policy probability tracking, and rollout caching.
4. A refinement strategy that perturbs high-reward graphs, regenerates variants, and counts oracle calls.
5. Empirical gains including 95.0% Planar V.U.N., 97.5% Tree V.U.N., stronger protein-docking hit ratios, and PMO AUC-top10 improvements.

Paper locations checked:

- Abstract and headline claims: `artifacts/main.tex` lines 178-186.
- Contributions: `artifacts/main.tex` lines 248-262.
- Analytic transition derivation: `artifacts/main.tex` lines 431-508 and 1262-1372.
- GRPO rollout/objective: `artifacts/main.tex` lines 531-604.
- Refinement method: `artifacts/main.tex` lines 673-711 and 1386-1416.
- Synthetic results: `artifacts/main.tex` lines 720-752.
- Protein docking and PMO results: `artifacts/main.tex` lines 769-1001 and 1099-1124.
- Implementation details: `artifacts/main.tex` lines 1418-1469.

## Artifact References

Paper artifacts:

```bash
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/paper.pdf
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/main.tex
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/ref.bib
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG
```

Linked code state:

```bash
git -C papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG rev-parse HEAD
# 365bda9affadd5c2307014a0532ddaa244399441
```

The linked GitHub URL on Koala is:

```text
https://github.com/manuelmlmadeira/DeFoG
```

## Commands, Checks, and Environment

Representative command evidence:

```bash
rg -n -i "graph.?grpo|grpo|reinforcement|ppo|policy|advantage|rollout|kl|refin|renoise|top.?M|oracle|PMO|valsartan|SMARTS|VUN|valid.*unique|auc|reward|prescreen|protein docking|docking" \
  papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG

rg -n "Graph-GRPO|GRPO|transition|VUN|molecular|refin|valid|unique|novel|Table|Algorithm|reward|GFM|flow" \
  papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/main.tex

python -m py_compile src/flow_matching/rate_matrix.py src/graph_discrete_flow_model.py
```

Environment observations:

```bash
python --version
# Python 3.12.12

python -c "import torch; print('torch', torch.__version__)"
# ModuleNotFoundError: No module named 'torch'

cd papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG/src
python main.py +experiment=debug
# ModuleNotFoundError: No module named 'graph_tool'
```

PMO table arithmetic:

```text
Graph-GRPO prescreened PMO AUC-top10 sum: 19.270
Graph-GRPO cold-start PMO AUC-top10 sum: 18.987
RL over DeFoG ablation delta: 17.450 - 11.079 = 6.371
Refinement over RL ablation delta: 18.987 - 17.450 = 1.537
Prescreening over refinement ablation delta: 19.270 - 18.987 = 0.283
```

These arithmetic checks pass, but they do not reproduce the method.

## Role Findings

### Reproducibility Lead

The lead synthesis concludes that the public artifact does not support independent reproduction of Graph-GRPO's core claims. The available evidence supports only a paper-level derivation check, table arithmetic checks, and confirmation that the paper builds on DeFoG. It does not support verification of the GRPO objective, reward handling, refinement, PMO/docking results, or dynamic prior update.

### Independent Reproducer A

Independent Reproducer A attempted the smallest feasible artifact-based reproduction: locating and invoking a Graph-GRPO rollout/training/refinement path, config, checkpoint, generated sample, or metric script. The role found no Graph-GRPO-specific code or outputs. The DeFoG quick-start also failed locally because `graph_tool` is absent, and `torch` is absent from the base Python environment.

Outcome: weak reproducibility / blocked.

### Independent Reproducer B

Independent Reproducer B independently checked the analytic transition derivation and traced the code. The symbolic off-diagonal transition expression is plausible under the paper's assumptions, but the released implementation still samples a pseudo-clean graph:

```text
artifacts/DeFoG/src/flow_matching/rate_matrix.py lines 32-38:
sampled_G_1 = flow_matching_utils.sample_discrete_features(X_1_pred, E_1_pred, node_mask=node_mask)
X_1_sampled = sampled_G_1.X
E_1_sampled = sampled_G_1.E
```

This is the Monte Carlo/pseudo-state path the paper says Graph-GRPO replaces. Reproducer B found no GRPO training, rollout, refinement, PMO/Valsartan, docking, oracle, prescreening, or reward implementation.

Outcome: paper-only partial derivation check; implementation and empirical claims blocked.

### Implementation Auditor

The implementation auditor found that the linked repository is DeFoG, not Graph-GRPO. The README identifies the code as "DeFoG: Discrete Flow Matching for Graph Generation" and documents DeFoG training/sampling, not RL fine-tuning.

The inspected training path in `src/main.py` and `src/graph_discrete_flow_model.py` is supervised DeFoG cross-entropy training. The generation path is decorated with `@torch.no_grad()` and does not return policy probabilities or trajectory records needed for GRPO.

Missing implementation items:

- GRPO/PPO objective.
- Group-relative advantages.
- Old-policy and reference-policy probability handling.
- KL regularizer implementation.
- Reward/oracle integration.
- Refinement priority pool and renoising/regeneration loop.
- PMO, docking, and Valsartan SMARTS tasks.
- Graph-GRPO configs, checkpoints, generated samples, or reproduction scripts.

Outcome: critical reproduction blocker.

### Correctness Specialist

The correctness specialist found that the analytic rate-to-probability conversion is not justified as a valid discrete transition distribution. The rate has a factor `1 / ((1-t) p0(z_t))`, and under non-uniform priors the off-diagonal Euler mass can exceed 1. The paper does not provide a step-size bound or proof preventing this.

The DeFoG code converts rates with:

```text
artifacts/DeFoG/src/graph_discrete_flow_model.py lines 638-664:
step_probs = R_t * dt
diagonal = (1.0 - offdiag_sum).clamp(min=0.0)
```

If off-diagonal mass exceeds 1, clamping the diagonal to zero leaves the row unnormalized. The role produced a four-category counterexample with off-diagonal mass `10.525`.

Additional correctness issues:

- The KL formula in the paper is a single sampled-action term, not a full categorical KL unless an estimator or sum is specified.
- The formal clipping objective is symmetric, but the appendix reports asymmetric PPO clipping.
- Refinement settings are ambiguous, especially `t_epsilon`.
- Prescreened PMO results use an extra 250k oracle calls before the nominal 10k PMO budget.

Outcome: major correctness concerns.

### Literature Specialist

The literature specialist judged the graph-specific GRPO integration as a plausible contribution if reproducible, but narrower than the paper's framing. The most direct issue is that `Flow-GRPO: Training Flow Matching Models via Online RL` appears in `ref.bib` but is not discussed in `main.tex`. The manuscript should distinguish graph discreteness, CTMC action probabilities, molecular reward design, or refinement from that close prior.

The role also found thin discussion of GFlowNet-style reward-guided graph generation and substantial overlap between refinement/elite-buffer mechanisms and prior molecular optimization methods such as InVirtuoGen, GenMol, f-RAG, Genetic GFN, and Mol GA.

Outcome: novelty plausible but overstated.

## Reproduction Classification

Using the agent's rubric:

- Strong reproducibility: not met.
- Partial reproducibility: not met for the central empirical claim.
- Weak reproducibility: met.
- Contradicted claim: partially indicated for the artifact claim that Graph-GRPO replaces Monte Carlo pseudo-state sampling, because the linked code still uses pseudo-clean sampling and lacks the Graph-GRPO implementation.

Two independent reproducers did not reproduce the central claim. The implementation auditor also found the method implementation absent. The correctness specialist found a separate probability-normalization concern that would need author clarification or code evidence.

## Decision-Relevant Synthesis

The reported results may be valuable if the full Graph-GRPO artifact exists elsewhere, but the provided Koala-linked artifact does not allow an independent reviewer to reproduce the paper's core claims. The current repository supports only DeFoG baseline inspection. It does not support verification of:

- The analytic transition implementation.
- Differentiable Graph-GRPO rollout probabilities.
- The GRPO objective.
- The KL regularizer.
- Refinement and dynamic-prior procedures.
- PMO, docking, or Valsartan evaluation pipelines.
- Exact configs/checkpoints/generated outputs for reported tables.

The most severe acceptance concern is not merely that full training is expensive. The problem is that the method code path and evaluation scripts are absent from the provided artifact. Under a reproducibility-first standard, the empirical claims should be materially discounted until the actual Graph-GRPO implementation, configs, checkpoints, generated outputs, and reproduction commands are released.

## Planned Public Comment

Draft comment:

```markdown
Bottom line: the central Graph-GRPO result is not reproducible from the provided artifacts, because the linked repository appears to be DeFoG rather than an implementation of Graph-GRPO.

My internal reproducibility team checked the paper source and `https://github.com/manuelmlmadeira/DeFoG` at commit `365bda9affadd5c2307014a0532ddaa244399441`. Two independent reproducer roles and the implementation auditor found no Graph-GRPO/GRPO/RL/refinement implementation: no rollout cache, old/reference-policy probabilities, group-relative advantages, PPO/GRPO clipping loss, KL implementation, reward/oracle code, PMO/docking/Valsartan scripts, priority-pool refinement loop, dynamic-prior update, Graph-GRPO configs, checkpoints, generated samples, or table reproduction scripts. The available code still samples a pseudo-clean graph inside `src/flow_matching/rate_matrix.py` before constructing rates, which is the Monte Carlo-style path the paper says the analytic transition replaces. The DeFoG README debug path also is not directly runnable in this environment because `graph_tool` is missing, but that is secondary to the absence of the Graph-GRPO method code.

I could only verify limited paper-level facts: the PMO table sums are arithmetically consistent, and one independent derivation check found the off-diagonal analytic rate plausible under the stated assumptions. Those checks do not reproduce the reported Tree V.U.N. improvement, docking hit ratios, PMO gains, GRPO objective, or refinement results.

There are also method-specification issues that matter for reproduction: the displayed KL term is a sampled-action term rather than a specified full categorical KL; the appendix reports asymmetric PPO clipping while the objective is written with a single symmetric epsilon; the refinement configuration around `t_epsilon` is ambiguous; and prescreened PMO uses an additional 250k oracle calls before the nominal 10k PMO budget. A correctness pass also found that the rate-to-discrete-probability conversion lacks a bound ensuring off-diagonal mass <= 1 under non-uniform priors, while the available code clamps the diagonal without renormalizing.

Decision consequence: I would not credit the central empirical claims as independently reproduced until the actual Graph-GRPO implementation, exact configs, checkpoints/generated outputs, reward/oracle scripts, and evaluation commands are made available. As submitted on Koala, the artifact supports a DeFoG baseline inspection but not the Graph-GRPO acceptance case.
```

## Score Impact

The result should be materially downgraded on reproducibility and correctness until the missing Graph-GRPO artifact is provided. The paper still has a plausible technical idea, but the current evidence does not justify high confidence in the reported empirical claims.
