# Independent Reproducer A Report

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Assigned role: Independent Reproducer A
Date: 2026-04-26

## Task Scope

Attempt a clean-room reconstruction of the central empirical result from the official paper text and artifacts: FlyGM should train as a connectome-structured policy and outperform graph-topology baselines in flybody locomotion.

## Evidence Examined

- `get_paper(a14d3e5d...)`: no GitHub repository in metadata.
- Official source archive downloaded from Koala.
- `main.tex`, especially architecture/training sections and appendix details.
- Project page `https://lnsgroup.cc/research/FlyGM/`, rendered with web access.

## Commands and Checks

```bash
mkdir -p papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts
curl -fsSL https://koala.science/storage/tarballs/a14d3e5d-d2c7-4877-bd15-45ee26effb81.tar.gz \
  -o papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source.tar.gz
tar -tzf papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source.tar.gz
rg -n "github|GitHub|code|repository|PPO|seed|random|rewir|baseline|MLP|learning rate|batch|steps|epochs|episodes" \
  papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source/main.tex
curl -fsSL https://lnsgroup.cc/research/FlyGM -o papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/flygm.html
rg -o "https?://[^\"'<> )]+" papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/flygm.html | sort -u
```

Observed artifact inventory:

- Source archive: `00README.json`, `main.tex`, `refs.bib`, ICML style files, and figures.
- No Python package, no notebooks, no configs, no environment YAML, no checkpoints, no training logs, no data manifests.
- Project page renders "Code (Coming soon)" and did not expose a GitHub repository URL in extracted links.

## Reimplementation Feasibility

Blocked. The method is described at a conceptual level, but a faithful implementation requires decisions not recoverable from the paper:

- Exact FlyWire data file, version, cleaning/filtering, neuron subset, edge thresholding, and neurotransmitter polarity mapping.
- Exact partitioning into afferent/intrinsic/efferent sets.
- Whether the model used signed synaptic counts (`main.tex:118-122`) or an unweighted graph (`main.tex:376` and project page text).
- Exact architecture of the encoder, gating map, update MLP, decoder, and value function.
- Whether hidden states are reset, carried between environment steps, detached through time, or truncated.
- Expert trajectory generation, rollout count, episode filtering, train/validation split, and successful-episode criterion.
- PPO hyperparameters beyond broad mention of clipping/value/entropy objective: clip epsilon, gamma, GAE lambda, rollout horizon, minibatches, epochs, entropy/value coefficients, normalization, reward definitions, and seeds.
- Metric implementation for position and angle errors in Table 1.

## Observed Result

I could not run the central experiment or reconstruct a faithful implementation from the available materials. The released information supports only a high-level pseudocode implementation with many unstated degrees of freedom.

Small derivation from paper/website dimensions:

```bash
awk 'BEGIN {
  printf "state_values_139246x32=%d\n", 139246*32;
  printf "efferent_flatten_1488x32=%d\n", 1488*32;
  printf "vision_added_dims=2*16*16=%d; 741+512=%d\n", 2*16*16, 741+2*16*16
}'
```

Output:

```text
state_values_139246x32=4455872
efferent_flatten_1488x32=47616
vision_added_dims=2*16*16=512; 741+512=1253
```

This confirms the reported visual augmentation dimension but also shows why exact memory/state handling matters for reimplementation.

## Match Status

Blocked. No reported metric, figure, or training curve can be independently recovered.

## Concrete Failure Reason

The paper promises a complete codebase/data/config repository, but the repository is not available. The paper alone omits the exact preprocessing, training, seed, and metric details required to reproduce the empirical claim.

Confidence level: high.
