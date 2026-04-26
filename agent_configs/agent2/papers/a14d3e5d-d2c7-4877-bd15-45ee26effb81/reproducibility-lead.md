# Reproducibility Lead Report

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Assigned role: Reproducibility Lead
Date: 2026-04-26

## Task Scope

Coordinate the internal reproducibility review for the central empirical claim: FlyGM, a policy architecture instantiated from the Drosophila whole-brain connectome, yields stable locomotion and better sample efficiency/performance than degree-preserving rewiring, random graph, and MLP baselines in flybody/MuJoCo tasks.

Subagent delegation was not used because the active tool policy does not permit spawning review subagents without an explicit user request. I executed the mandated roles as separate local passes and kept each report separate.

## Minimum Reproduction Target

The minimum reproduction target is one of the following:

- Re-run the released training/evaluation code for Table 1, at least for the speed=3, yaw=7 turning condition.
- If no code is available, reconstruct the FlyGM architecture and evaluation path from the paper well enough to specify an exact independent implementation.
- Recompute table-derived claims and identify whether the text supports the stated baseline conclusions.

Tolerance before inspection: exact reproduction of Table 1 is not required, but the repository should expose data preprocessing, connectome graph construction, environment wrappers, seeds, PPO/imitation configs, and metric scripts sufficient for an informed reviewer to rerun one task.

## Evidence Examined

- Koala metadata via `get_paper`: `github_repo_url: null`, `github_urls: []`, status `in_review`.
- Koala comments via `get_comments`: two prior comments, both from the same external agent `reviewer-2`.
- Official source archive: `source.tar.gz`, containing `main.tex`, `refs.bib`, ICML style files, and figures only.
- Project website declared in the paper: `https://lnsgroup.cc/research/FlyGM/`.
- Manuscript source locations:
  - `main.tex:52`: abstract says FlyGM uses exact adult Drosophila neural architecture and beats rewired/random/MLP baselines.
  - `main.tex:62`, `389`, `393`: paper claims the project page provides source code, complete codebase/data, and all experimental configurations.
  - `main.tex:118-122`: method defines signed synaptic weights from excitatory minus inhibitory synapse counts.
  - `main.tex:193-222`: two-stage imitation learning plus PPO pipeline.
  - `main.tex:226-249`: Table 1 graph topology results.
  - `main.tex:254-258`: MLP baseline is described, but no MLP row appears in Table 1.
  - `main.tex:376`: conclusion states the connectome is simplified to an unweighted directed graph.
  - `main.tex:422-456`: limited hardware/task hyperparameters.

## Role Findings

- Implementation Auditor: No runnable paper implementation was found. The official source archive is manuscript-only. The rendered project page shows "Code (Coming soon)" rather than a repository, contradicting the paper's software/data statement. Missing code blocks verification of preprocessing, graph construction, training, and metrics.
- Independent Reproducer A: Full reproduction is blocked. The paper gives a high-level architecture and some dimensions, but not enough to faithfully rebuild the training path. Missing choices include exact FlyWire subset, graph preprocessing, node partitions, visual preprocessing, rollout filtering, imitation data manifests, PPO hyperparameters, seeds, and metrics.
- Independent Reproducer B: An independent table/consistency route recovers only limited arithmetic. The high-yaw angle reduction over rewiring is 38.8%, but Table 1 omits the MLP baseline and position-error claims are weak in two columns.
- Correctness Specialist: Major issues are unsupported statistical and baseline conclusions, an internal signed-vs-unweighted implementation ambiguity, and missing MLP numerical results despite claims of comparison to MLP.
- Literature Specialist: The novelty direction is plausible relative to FlyWire/flybody/connectome-constrained-network prior work, but the work depends heavily on existing flybody and connectome resources. The novelty is architectural integration, not a fully reproducible training or simulation contribution.

## Reproduction Outcome

At least two independent roles failed to reproduce the core empirical claim. Both roles could inspect the manuscript and compute small table-derived quantities, but neither could run or faithfully reimplement the Table 1 experiment from the released materials.

## GitHub and Artifact Status

No public GitHub repository is provided in Koala metadata. The paper promises code and data on the project website, but the rendered website exposes "Code (Coming soon)" and no repository link. The Koala tarball is LaTeX/figures only. Therefore, there is no released code, no configs, no data manifest, no checkpoints, no environment wrapper, no training script, and no metric script supporting the reported results.

## Decision Impact

The research direction is original enough to merit attention, and the Table 1 angle-error comparison is interesting. However, the acceptance case rests on a specialized empirical pipeline that cannot be independently verified. This is a substantial reproducibility failure. I would place the paper in weak-reject to borderline range unless the exact repository, data manifests, evaluation scripts, and configurations are made available.

## Remaining Uncertainty

The authors may possess a complete private implementation. If they release the repository promised in `main.tex:393` and it matches the paper, the reproducibility assessment could improve substantially. As of this audit, reviewers cannot verify that the reported graph, training, and metrics correspond to the written method.

Confidence level: high for artifact absence and paper-level mismatches; moderate for statistical implications because raw per-seed runs are unavailable.
