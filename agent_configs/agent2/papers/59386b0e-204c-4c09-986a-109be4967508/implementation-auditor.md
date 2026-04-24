# Implementation Auditor Report

Paper: `59386b0e-204c-4c09-986a-109be4967508`, "Graph-GRPO: Training Graph Flow Models with Reinforcement Learning"
Role: Implementation Auditor
Artifact audited: `artifacts/DeFoG` at commit `365bda9affadd5c2307014a0532ddaa244399441`

## Bottom Line

The linked DeFoG repository does not implement Graph-GRPO. It is a public DeFoG training and sampling codebase for supervised discrete flow matching, with no released GRPO/RL fine-tuning loop, no reward/oracle code, no rollout cache, no old-policy/reference-policy probability ratio objective, no refinement priority pool, and no PMO or docking evaluation pipeline. The code also still samples a pseudo clean graph inside the rate-matrix computation, directly contradicting the paper's central claim that Graph-GRPO replaces Monte Carlo pseudo-state sampling with an analytic differentiable transition probability for RL training.

This is a severe reproduction blocker for the paper's core claims. The released artifact can support only the claim that the authors build on a DeFoG-like backbone and sampling/evaluation infrastructure; it cannot independently verify the reported Graph-GRPO training, refinement, molecular optimization, docking, or dynamic-prior results.

## Artifact Inventory

Audited files and directories:

- Paper source: `artifacts/main.tex`, `artifacts/ref.bib`, `artifacts/paper.pdf`.
- Repository: `artifacts/DeFoG`, 82 non-git files.
- Top-level repo files: `README.md`, `requirements.txt`, `environment.yaml`, `Dockerfile`, `setup.py`.
- Configs: `configs/config.yaml`, `configs/train/train_default.yaml`, `configs/sample/sample_default.yaml`, `configs/general/general_default.yaml`, `configs/experiment/{planar,tree,zinc,guacamol,moses,qm9_no_h,qm9_with_h,sbm,comm20,tls}.yaml`, dataset configs.
- Source tree: `src/main.py`, `src/graph_discrete_flow_model.py`, `src/flow_matching/{rate_matrix.py,flow_matching_utils.py,noise_distribution.py,time_distorter.py,utils.py}`, dataset loaders, metrics, transformer model, visualization, ORCA evaluator.

Commands used:

```bash
git -C papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG rev-parse HEAD
find papers/59386b0e-204c-4c09-986a-109be4967508/artifacts/DeFoG -maxdepth 4 -type f | sort
rg -n -i "graph[-_ ]?grpo|grpo|ppo|reinforcement|reward|advantage|rollout|trajectory|old_policy|old policy|ratio|clip|kl|refine|refinement|perturb|transition|rate_matrix|posterior|qed|oracle|guacamol|moses|vina|smarts|scaffold|lead|hit" artifacts/DeFoG artifacts/main.tex
sed -n '1,260p' artifacts/DeFoG/README.md
sed -n '1,320p' artifacts/DeFoG/src/main.py
sed -n '1,980p' artifacts/DeFoG/src/graph_discrete_flow_model.py
sed -n '1,420p' artifacts/DeFoG/src/flow_matching/rate_matrix.py
sed -n '430,1562p' artifacts/main.tex
```

No future or post-publication sources about the exact paper were used. I inspected only the provided artifacts and the linked DeFoG clone.

## Paper Claims Tested

The implementation audit focused on the paper's implementation-dependent claims:

1. Graph-GRPO is an online RL framework for training graph flow models under verifiable rewards.
2. The method replaces non-differentiable Monte Carlo pseudo-graph sampling with an analytic transition probability suitable for differentiable RL.
3. The implementation collects grouped rollouts, caches transition probabilities, computes group-relative advantages, and optimizes a clipped GRPO/PPO-style objective with KL regularization.
4. The implementation supports refinement by renoising high-reward candidates, regenerating variants, and retaining a top-M priority pool.
5. The released repository supports the reported synthetic graph, protein docking, and PMO target-property experiments.
6. The implementation supports the appendix settings: AdamW with reward-plateau LR decay, effective batch size 200 via accumulation, gradient clipping 5.0, group size K=60, asymmetric clipping, advantage clipping, KL coefficient beta=0.005, dynamic prior update, top-1000 reward buffer, and oracle budgets/refinement schedule.

## Code Paths Inspected

### README and commands

`README.md` identifies the repository as "DeFoG: Discrete Flow Matching for Graph Generation" and as a "PyTorch implementation of the DeFoG model for training and sampling discrete graph flows" (`README.md:1-3`). The only usage documented is supervised DeFoG training:

```bash
python main.py +experiment=<dataset> dataset=<dataset>
```

and checkpoint sampling/evaluation with `general.test_only` and `sample.{eta,omega,time_distortion}` (`README.md:70-137`). There is no Graph-GRPO command, RL fine-tuning command, reward/oracle command, rollout command, refinement command, PMO command, docking command, or dynamic-prior command.

The README's checkpoint section points to DeFoG checkpoints and generated samples, not Graph-GRPO checkpoints (`README.md:156-162`). The "Upon request" section lists "protein / EGO datasets" and "FCD score for molecules" as not in the public repo (`README.md:166-170`), which is material because the paper's molecular optimization claims depend on protein docking and molecular evaluation infrastructure.

### Training entry point

`src/main.py` builds dataset modules, creates `GraphDiscreteFlowModel`, and either runs `trainer.fit(...)` or `trainer.test(...)` (`src/main.py:210-251`). It has no branch for RL fine-tuning, rollout generation, reward scoring, or policy/reference model management.

`GraphDiscreteFlowModel.training_step` applies DeFoG noise, predicts node/edge logits, and optimizes `self.train_loss(...)` against the clean graph labels (`src/graph_discrete_flow_model.py:118-158`). `TrainLossDiscrete` is explicitly cross entropy over node, edge, and y targets (`src/metrics/train_metrics.py:25-93`). This is supervised DeFoG pretraining, not a GRPO objective.

### Sampling and rate matrix

`sample_batch` is decorated with `@torch.no_grad()` and performs standard sampling from the prior/noise distribution through `sample_p_zs_given_zt` (`src/graph_discrete_flow_model.py:455-636`). Because it runs under `torch.no_grad()`, this path cannot be the differentiable rollout path claimed for RL training.

`sample_p_zs_given_zt` computes model predictions, obtains a rate matrix, converts rates to step probabilities, and samples the next graph state (`src/graph_discrete_flow_model.py:666-747`). It returns only the sampled one-hot/discrete graph state, not cached transition probabilities, per-step log-probabilities, old-policy probabilities, or any trajectory object needed by the GRPO objective.

Most importantly, `RateMatrixDesigner.compute_graph_rate_matrix` still samples a pseudo clean graph:

```python
sampled_G_1 = flow_matching_utils.sample_discrete_features(X_1_pred, E_1_pred, node_mask=node_mask)
X_1_sampled = sampled_G_1.X
E_1_sampled = sampled_G_1.E
```

This is at `src/flow_matching/rate_matrix.py:25-42`. It then computes `Rstar`, `RDB`, and `R_tg` using `X_1_sampled`/`E_1_sampled` (`src/flow_matching/rate_matrix.py:44-71`). The sampling utility uses multinomial sampling (`src/flow_matching/flow_matching_utils.py:53-92`). Therefore, the public implementation does not implement the paper's analytic expression from model probabilities alone, as claimed in `main.tex:491-508`.

### Configs and dependencies

The training configs are DeFoG configs:

- `configs/train/train_default.yaml`: `n_epochs: 1000`, `batch_size: 512`, `lr: 0.0002`, `weight_decay: 1e-12`, `clip_grad: null`.
- `configs/experiment/zinc.yaml`: supervised training with `n_epochs: 300`, `batch_size: 256`, `lr: 2e-4`, and sampling hyperparameters `eta: 300`, `omega: 0.1`.
- `configs/experiment/planar.yaml` and `tree.yaml`: DeFoG synthetic graph configs with long supervised training schedules and sampling hyperparameters.

These do not match the paper appendix's Graph-GRPO training configuration: LR `2e-5` decayed to `1e-5`, weight decay `1e-4`, physical batch size 5-40 with gradient accumulation to effective batch 200, gradient clip 5.0, group size K=60, asymmetric PPO clipping, advantage clipping, KL coefficient beta=0.005, dynamic prior update, and top-M/top-1000 buffers (`main.tex:1441-1468`).

`requirements.txt` and `environment.yaml` contain general DeFoG dependencies. They include RDKit, graph-tool, PyTorch, PyG, Lightning, and psi4, but no obvious PMO, docking/Vina, reward-oracle, or RL-specific dependency. The repository search found no implementation of PMO or docking reward terms from `main.tex:1537-1560`.

## Paper-to-Code Matches

The code does match a limited subset of the paper's backbone description:

- The paper says the framework builds on DeFoG and a Graph Transformer/RWSE backbone (`main.tex:914-918`, `main.tex:1430-1438`). The repo is DeFoG, with `GraphTransformer` in `src/models/transformer_model.py`, DeFoG data modules, and RRWP-related config/features.
- The paper uses synthetic datasets and molecular datasets also present in the DeFoG repo: Planar, Tree, SBM, Comm20, QM9, Guacamol, MOSES, ZINC.
- The repo can train and sample a DeFoG model and compute standard generation metrics; this plausibly supports reproducing a DeFoG baseline, not Graph-GRPO.
- `compute_step_probs` implements the discrete-time diagonal/off-diagonal conversion from rates to transition probabilities (`src/graph_discrete_flow_model.py:638-664`), which is related to the graph flow sampling machinery described in the paper. It does not provide the Graph-GRPO differentiable policy objective.

## Paper-to-Code Discrepancies

### 1. No Graph-GRPO implementation

Searches over all non-git repository files found no substantive `Graph-GRPO`, `GRPO`, `PPO`, `reinforcement`, `reward`, `advantage`, `policy ratio`, `pi_old`, `rollout`, `oracle`, `PMO`, `docking`, `Valsartan`, or `SMARTS` implementation. Matches were either absent or unrelated to DeFoG metrics/data.

The paper describes rollout collection, cached transition probabilities, group-normalized rewards, importance sampling ratios, clipped GRPO objective, KL to a reference policy, and reward maximization (`main.tex:527-604`). None of this exists in `src/main.py`, `src/graph_discrete_flow_model.py`, configs, or metrics.

### 2. The rate matrix still uses Monte Carlo pseudo-state sampling

The paper's central implementation claim is that Graph-GRPO replaces Monte Carlo pseudo-graph sampling with an analytic transition probability (`main.tex:180-183`, `main.tex:248-260`, `main.tex:491-508`). The provided code does the opposite: `compute_graph_rate_matrix` samples `sampled_G_1` from `X_1_pred,E_1_pred` and then conditions rate calculations on this sampled clean graph (`src/flow_matching/rate_matrix.py:25-42`). This is the conditional/Monte Carlo DeFoG-style path, not the analytic probability traversal claimed by Graph-GRPO.

### 3. Sampling is explicitly non-differentiable and no-gradient

The only rollout-like generation path is `sample_batch`, and it is decorated with `@torch.no_grad()` (`src/graph_discrete_flow_model.py:455`). It samples categorical states with `multinomial` (`src/flow_matching/flow_matching_utils.py:53-92`). This can evaluate or generate graphs, but it cannot serve as the differentiable rollout mechanism claimed for RL training.

### 4. Training is supervised cross-entropy, not RL

`training_step` returns a cross-entropy training loss on clean graph labels (`src/graph_discrete_flow_model.py:118-158`; `src/metrics/train_metrics.py:25-93`). There is no terminal reward, no per-trajectory reward, no group mean/std advantage, no clipping, no policy/reference model, and no KL regularizer in the training path.

### 5. No refinement implementation

The paper describes a priority pool of top-M high-reward graphs, renoising to `t_epsilon`, regenerating variants, evaluating rewards, and retaining the best candidates (`main.tex:693-708`, `main.tex:1457-1468`). The DeFoG repo has no such priority queue, top-M pool, reward buffer, `t_epsilon`, renoising-from-candidate path, oracle budget schedule, or refinement command. The only `refinement` matches in code are unrelated MCMC refinement steps in SBM graph validity checks inside `src/analysis/spectre_utils.py`.

### 6. No molecular optimization, PMO, or docking pipeline

The paper reports protein docking and PMO target-property optimization and defines QED/SA/novelty/docking rewards and Valsartan SMARTS curriculum (`main.tex:914-918`, `main.tex:1537-1560`). The repo includes standard molecular generation datasets and RDKit evaluation utilities, but no docking target files, docking scoring scripts, PMO oracle integration, Valsartan SMARTS objective, Sitagliptin property kernels, two-stage curriculum, or oracle-call accounting.

### 7. Appendix implementation settings are absent or contradicted

The public configs use DeFoG defaults such as LR `2e-4`, `batch_size: 256/512`, `clip_grad: null`, and no accumulation config. The paper claims reward-plateau scheduling, weight decay `1e-4`, gradient clipping 5.0, effective batch 200, K=60, clipping epsilons, beta=0.005, advantage clipping, dynamic prior update, top-1000 buffer, and refinement schedules (`main.tex:1441-1468`). None are implemented or configurable in the released code.

### 8. Conditional guidance path appears suspicious

In `sample_p_zs_given_zt`, the conditional branch recomputes an unconditional prediction after setting `noisy_data["y_t"] = uncond_y`, but then calls `compute_graph_rate_matrix(..., G_1_pred)` using the earlier conditional `G_1_pred`, not the newly computed unconditional `pred_X,pred_E` (`src/graph_discrete_flow_model.py:709-726`). This is not central to Graph-GRPO, but it is an implementation concern in the released DeFoG code: the nominal unconditional guidance rate matrix may not actually use unconditional predictions.

## Reproducibility Blockers

Blocking for core Graph-GRPO reproduction:

- No Graph-GRPO source code.
- No RL fine-tuning entry point or command.
- No reward/oracle implementations for synthetic rewards, PMO, docking, Valsartan SMARTS, or curriculum.
- No rollout buffer/cache, transition-probability cache, old-policy/reference-policy handling, or GRPO objective.
- No analytic transition-probability implementation matching Proposition 1; the released code samples a pseudo clean graph.
- No refinement implementation, priority pool, top-M selection, top-1000 buffer, dynamic prior update, or oracle budget logic.
- No released Graph-GRPO configs matching the appendix.
- No Graph-GRPO checkpoints or generated outputs in the provided artifact tree.
- No documented commands to reproduce the paper's Graph-GRPO tables/figures.

Secondary reproduction limitations for the DeFoG baseline:

- README points to external checkpoints/results on a Switch Drive link; these were not included in the provided artifact tree.
- Dataset loaders fetch data from external URLs at runtime.
- ORCA must be compiled manually.
- Environment uses older package versions and graph-tool/rdkit/psi4, which can be difficult to reproduce exactly, though this is secondary to the absence of Graph-GRPO itself.

## Severity for Acceptance Decision

Severity: critical.

The artifact does not implement the method named in the paper. It implements a DeFoG baseline and standard sampling/evaluation paths, while the paper's acceptance case rests on Graph-GRPO-specific RL training, analytic transition probabilities, refinement, docking/PMO rewards, and dynamic-prior mechanisms. Because those mechanisms are absent, the reported improvements over DeFoG, GDPO, PMO baselines, and molecular docking baselines cannot be independently reproduced from the released repository. The central empirical and implementation claims should be treated as weakly supported by artifacts unless the authors release the actual Graph-GRPO training/refinement/reward code and exact configs/checkpoints.
