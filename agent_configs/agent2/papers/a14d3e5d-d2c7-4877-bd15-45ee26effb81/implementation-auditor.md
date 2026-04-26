# Implementation Auditor Report

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Assigned role: Implementation Auditor
Date: 2026-04-26

## Task Scope

Adversarially inspect official artifacts and declared links to determine whether implementation, data preprocessing, configurations, training/evaluation scripts, seeds, and metric code support the reported results.

## Artifact Inventory

Koala metadata:

- `github_repo_url`: null
- `github_urls`: []
- `pdf_url`: `/storage/pdfs/a14d3e5d-d2c7-4877-bd15-45ee26effb81.pdf`
- `tarball_url`: `/storage/tarballs/a14d3e5d-d2c7-4877-bd15-45ee26effb81.tar.gz`

Source archive contents:

```text
00README.json
algorithm.sty
algorithmic.sty
fancyhdr.sty
figures/
icml2026.bst
icml2026.sty
main.tex
refs.bib
```

The source archive contains manuscript files and figures only. It does not contain a code repository, scripts, configs, data manifests, model checkpoints, logs, or environment specification.

## Commands Used

```bash
curl -fsSL https://koala.science/storage/tarballs/a14d3e5d-d2c7-4877-bd15-45ee26effb81.tar.gz \
  -o papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source.tar.gz
tar -tzf papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source.tar.gz
find papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source -maxdepth 3 -type f
rg -n "github|GitHub|code|repository|repo|dataset|PPO|seed|config|learning rate|batch|epochs" \
  papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source/main.tex
curl -fsSL https://lnsgroup.cc/research/FlyGM \
  -o papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/flygm.html
rg -o "https?://[^\"'<> )]+" papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/flygm.html | sort -u
```

## Paper-to-Artifact Matches

- The project page in `main.tex:62`, `389`, and `393` resolves and shows videos, paper link, and a code placeholder.
- The source archive matches a normal LaTeX submission package.
- The appendix includes partial implementation details: A100 80GB PCIe GPUs, Intel Xeon Gold 6348 CPUs, AdamW, ReduceLROnPlateau, gradient clipping, walking/flight input and action dimensions, learning rates, and message-passing width/layers.

## Paper-to-Artifact Discrepancies

Major discrepancy: the manuscript claims source code is available, but I did not find it.

- `main.tex:62`: figure caption states demonstration videos and source code are available at the project page.
- `main.tex:389`: accessibility section states the webpage provides demonstration videos and source code.
- `main.tex:393`: software/data section states the authors provide "complete codebase and data in an anonymous repository" and "all experimental configurations."
- The rendered project page shows "Code (Coming soon)" rather than a repository link.
- Koala metadata supplies no GitHub URL.
- Extracted project-page URLs include the arXiv link and video embeds, but no FlyGM code repository.

Second discrepancy: the graph-weighting story is inconsistent.

- `main.tex:118-122` defines signed synaptic weights as excitatory minus inhibitory synapse counts.
- `main.tex:376` states results hold "even when simplified to an unweighted directed graph without synapse counts or neurotransmitter types."
- The project page likewise describes the method as an unweighted directed graph.
- The released artifacts contain no code or configs to determine which graph was actually used in the reported Table 1 runs.

Third discrepancy: the MLP baseline is claimed but not reported in the main table.

- `main.tex:52`, `73`, and `254-258` say the method is compared against MLP baselines.
- Table 1 (`main.tex:226-249`) reports only connectome, degree-preserving rewiring, and Erdos-Renyi random graph. No MLP numerical row is available there.

## Missing Implementation Elements

The following are necessary for faithful reproduction and were not released:

- FlyWire graph extraction and versioned data manifest.
- Node filtering, edge thresholding, sign/weight preprocessing, and afferent/intrinsic/efferent partition files.
- flybody environment wrappers and the modifications adding binocular visual signals.
- Expert rollout generation scripts and successful-episode filtering code.
- Imitation-learning dataset manifests and splits.
- Complete architecture definitions for encoder, gate, update MLP, decoder, value network, recurrent state handling, and normalization.
- PPO configuration: seed list, rollout horizon, batch size, optimizer settings beyond learning rate, clip epsilon, GAE lambda, discount, value/entropy coefficients, number of epochs, and stopping criteria.
- Evaluation and metric scripts for Table 1 position and angle errors.
- Training logs, raw per-seed metrics, and checkpoints.

## Reproducibility Blockers

Severity: major to severe.

The core empirical result cannot be audited. An informed reviewer cannot tell whether the reported controller used signed weights, unweighted topology, a particular filtered connectome, a particular visual-input modification, or an unreleased environment wrapper. The code absence is especially serious because the paper itself promises complete code, data, and configurations.

## Acceptance Consequence

This is not a minor packaging issue. The contribution is a specialized empirical system whose credibility depends on exact graph construction, environment adaptation, and RL/imitation settings. Without the repository, the reported performance should be treated as unverified.

Confidence level: high.
