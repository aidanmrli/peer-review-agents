# Reproducibility Lead Report

Paper: `59386b0e-204c-4c09-986a-109be4967508`

Title: "Graph-GRPO: Training Graph Flow Models with Reinforcement Learning"

Role: Reproducibility Lead

Date: 2026-04-24

## Claim Being Tested

The central claim tested by the team is that Graph-GRPO provides a reproducible online RL framework for graph flow models by:

1. Replacing DeFoG-style Monte Carlo pseudo-clean-state sampling with an analytic transition probability suitable for policy-gradient training.
2. Implementing GRPO rollouts, group-relative advantages, clipped policy-ratio optimization, and KL regularization.
3. Implementing a refinement procedure that renoises high-reward graphs, regenerates variants, and accounts for oracle calls.
4. Producing the reported empirical gains on synthetic graph generation, protein docking, and PMO molecular optimization.

The paper locations checked include `artifacts/main.tex` lines 178-186, 248-262, 431-508, 531-604, 673-711, 720-752, 962-1001, 1099-1124, 1262-1372, and 1418-1469.

## Internal Review Protocol

Five independent role reports were produced before any public comment:

- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

I used those role reports, the paper source, and the linked code artifact as the evidence base for this lead synthesis. No future information about the exact submission was used.

## Artifacts and Environment

Workspace:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent2
```

Paper artifacts:

```bash
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/paper.pdf
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/main.tex
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/ref.bib
papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG
```

Linked code artifact:

```bash
git -C papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG rev-parse HEAD
# 365bda9affadd5c2307014a0532ddaa244399441
```

Local environment observations:

```bash
python --version
# Python 3.12.12

python -c "import torch; print(torch.__version__)"
# ModuleNotFoundError: No module named 'torch'
```

Independent Reproducer A also attempted the DeFoG README debug entry point:

```bash
cd papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG/src
python main.py +experiment=debug
# ModuleNotFoundError: No module named 'graph_tool'
```

This environment blocker matters less than the artifact blocker: the available repository appears to be DeFoG, not Graph-GRPO.

## Commands Used Across Roles

Representative commands from the team reports:

```bash
rg -n -i "graph.?grpo|grpo|reinforcement|ppo|policy|advantage|rollout|kl|refin|renoise|top.?M|oracle|PMO|valsartan|SMARTS|VUN|valid.*unique|auc|reward|prescreen|protein docking|docking" \
  papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG

rg -n "Graph-GRPO|DeFoG|flow matching|discrete flow|GFlowNet|reinforcement|preference|reward|molecular|baseline|related|prior|novel|contribution|GRPO|RL|fine" \
  papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/main.tex \
  papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/ref.bib

python -m py_compile src/flow_matching/rate_matrix.py src/graph_discrete_flow_model.py
```

PMO table arithmetic was checked by summing the reported table entries. The prescreened Graph-GRPO PMO AUC-top10 values sum to `19.270`, and the cold-start values sum to `18.987`, matching the paper.

## Role-by-Role Findings

### Independent Reproducer A

Outcome: weak reproducibility / blocked.

Reproducer A attempted a lightweight reproduction by locating a Graph-GRPO training, rollout, refinement, checkpoint, generated-sample, or metric path. The role found no Graph-GRPO-specific implementation, no GRPO/RL/refinement code, no PMO/docking scripts, no generated outputs, and no matching experiment configs. The DeFoG debug command also failed in the local environment due to missing `graph_tool`, while `torch` was absent.

The only positive check was table arithmetic consistency for the PMO totals and ablation deltas. This does not reproduce the method.

### Independent Reproducer B

Outcome: partial paper-only derivation check; empirical and code reproduction blocked.

Reproducer B independently checked the analytic transition derivation and found the off-diagonal symbolic form plausible under the paper's assumptions. However, code tracing showed that `artifacts/DeFoG/src/flow_matching/rate_matrix.py` still samples a pseudo-clean graph from model predictions before constructing the rate matrix. This is the DeFoG-style sampled route, not the analytic expectation claimed for Graph-GRPO.

The role also found no GRPO training, refinement, PMO/Valsartan, docking, reward, oracle, prescreening, or rollout implementation. It identified table-framing issues in the synthetic graph results, including inconsistent DeFoG/Graph-GRPO V.U.N. comparisons across main and appendix tables.

### Implementation Auditor

Outcome: critical implementation blocker.

The implementation auditor concluded that the linked repository is a public DeFoG implementation rather than a Graph-GRPO implementation. The README describes "DeFoG: Discrete Flow Matching for Graph Generation"; documented commands are supervised DeFoG training and sampling commands. The code path inspected in `src/main.py`, `src/graph_discrete_flow_model.py`, and `src/flow_matching/rate_matrix.py` implements DeFoG training/sampling, not online RL.

Major missing implementation components:

- GRPO/RL fine-tuning loop.
- Reward/oracle code.
- Rollout cache and old-policy/reference-policy probability handling.
- Group-relative advantage computation.
- PPO/GRPO clipping and KL regularization.
- Refinement priority pool and renoising/regeneration loop.
- PMO, Valsartan SMARTS, and protein docking pipelines.
- Graph-GRPO configs, checkpoints, generated samples, and table reproduction scripts.

The auditor also noted a suspicious conditional-guidance path in DeFoG code, but that issue is secondary to the absence of Graph-GRPO.

### Correctness Specialist

Outcome: major correctness concerns.

The correctness specialist found a technical weakness in the conversion of continuous-time analytic rates to discrete transition probabilities. The rate expression contains a factor `1 / ((1-t) p0(z_t))`; for non-uniform priors or near final time, the off-diagonal Euler mass can exceed 1. The paper does not provide a step-size bound or proof that the selected schedule avoids this.

The released DeFoG code multiplies rates by `dt`, zeros the diagonal, and clamps the diagonal to at least zero in `src/graph_discrete_flow_model.py` lines 638-664. If off-diagonal mass already exceeds 1, this clamp prevents a negative diagonal but does not renormalize the row. The role gave a concrete four-category counterexample where the off-diagonal mass is `10.525`.

Additional correctness issues:

- The displayed KL term is a single sampled-action term, not a full categorical KL divergence unless a sum or estimator is specified.
- The formal clipping objective uses symmetric `epsilon`, while the appendix reports asymmetric PPO clipping.
- The refinement configuration is ambiguous: the ablation tests `t_epsilon` values excluding `0.8`, while the appendix states fixed `t_epsilon=0.8`.
- Prescreened PMO results consume 250k oracle calls before the nominal 10k optimization budget and should not be framed as equal-budget state of the art.
- Some synthetic graph conclusions are metric-specific rather than uniformly dominant.

### Literature Specialist

Outcome: plausible but overstated novelty.

The literature specialist judged the core graph-specific RL integration as plausible incremental-to-moderate novelty if the method and experiments reproduce. The strongest literature concern is that `Flow-GRPO: Training Flow Matching Models via Online RL` appears in `ref.bib` but is not discussed in the manuscript text. The paper should explicitly distinguish its contribution from Flow-GRPO if that prior was available by release.

Other framing weaknesses:

- GFlowNet-style reward-guided graph generation is under-discussed for a reward-guided graph generator paper.
- Refinement, elite buffers, local modification, retrieval, fragment replacement, and genetic search are already central in molecular optimization prior work.
- Prescreened state-of-the-art claims need setting-specific qualification because of the additional 250k oracle calls.

## Reproduction Outcome

Overall outcome: weak reproducibility.

Two independent reproducers failed to reproduce the central empirical claim from the provided artifacts. Both found that the linked repository does not expose the Graph-GRPO training/refinement/reward implementation needed to verify the reported results. One role partially validated the paper's symbolic off-diagonal transition derivation, but that does not validate the method implementation or empirical claims.

The strongest reproducible evidence is limited to:

- Paper-only symbolic plausibility of the analytic off-diagonal rate under stated assumptions.
- Internal arithmetic consistency of the PMO table totals and ablation deltas.
- Existence of a DeFoG backbone repository compatible with the paper's claim that it builds on DeFoG.

The central empirical claims are not independently reproducible from the released artifacts.

## Score Impact

The paper's tables may be promising if the authors can supply the actual Graph-GRPO implementation, configs, checkpoints, generated outputs, and evaluation scripts. Under the current artifact state, however, a reproducibility-first review must materially discount the reported gains. The missing method implementation, possible invalid transition-probability rows, under-specified KL/objective details, and ambiguous refinement/oracle accounting are decision-relevant weaknesses.

My recommendation for the public discussion is a top-level reproducibility comment that asks for the missing Graph-GRPO artifact and clearly states that the central claim is not currently reproducible under the provided setup.
