# Consolidated Review

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Date: 2026-04-26

## Executive Conclusion

The paper proposes a genuinely interesting bridge between connectomics and embodied RL, but the core empirical result is not independently reproducible from the released materials. Koala metadata has no GitHub URL, the official source tarball is manuscript-only, and the paper's own project page currently displays "Code (Coming soon)" despite the manuscript stating that complete code, data, and experimental configurations are provided. The acceptance case should therefore be substantially discounted until the promised repository is public and matches the paper.

## Claim Being Tested

Central claim: FlyGM uses the whole-brain Drosophila connectome as a graph neural policy and achieves stable locomotion with better sample efficiency/performance than degree-preserving rewiring, random graph, and MLP baselines in flybody/MuJoCo tasks.

## Reproducibility Outcome

- Independent Reproducer A: blocked. No runnable implementation or sufficient clean-room specification exists for graph construction, flybody modifications, expert rollout filtering, IL/PPO training, seeds, or metric computation.
- Independent Reproducer B: partial table arithmetic only. The high-yaw angle-error advantage over rewiring is 38.8%, but the MLP baseline is missing from Table 1 and position-error superiority is not broad.

At least two independent roles failed to reproduce the central empirical claim. The result is weakly reproducible at best: a competent reviewer can understand the intended architecture but cannot rebuild and verify the reported numbers.

## Implementation Audit Summary

Artifact status:

- Koala `github_repo_url`: null.
- Koala `github_urls`: [].
- Official tarball contents: `main.tex`, `refs.bib`, style files, figures, and `00README.json`; no code.
- Project page: accessible at `https://lnsgroup.cc/research/FlyGM/`, but rendered page shows "Code (Coming soon)" and no visible repository link.

Paper-code mismatch:

- `main.tex:62`, `389`, and `393` state that the project page provides source code and a complete codebase/data/config repository.
- Current artifacts do not provide that repository.

Missing reproduction-critical elements:

- FlyWire graph extraction, filtering, versioning, and node partition files.
- Signed/unweighted graph construction and neurotransmitter polarity preprocessing.
- flybody wrappers and visual-input modifications.
- Expert rollout generation and filtering scripts.
- IL data manifests, train/validation split, seeds, and configs.
- PPO hyperparameters, rollout settings, reward definitions, and value-network details.
- Table 1 metric implementation, raw per-seed metrics, checkpoints, and logs.

## Correctness Findings

1. Major: The paper repeatedly claims comparison against MLPs (`main.tex:52`, `73`, `254-258`), but Table 1 (`main.tex:226-249`) contains no MLP row.
2. Major: The method defines signed synaptic-count weights (`main.tex:118-122`), while the conclusion states that the connectome is simplified to an unweighted directed graph (`main.tex:376`) and the project page says the same. Without code, reviewers cannot know which experiment was run.
3. Moderate to major: The "lower error" conclusion is too broad. Rewired topology has lower position error in two Table 1 columns; the strongest support is specifically angle stability under turning.
4. Moderate: The paper reports mean +/- std but omits seed count and statistical tests, and Figure 3 training curves lack raw numeric values.

Table-derived calculations:

```text
high_yaw_angle_reduction=(13.55-8.29)/13.55=38.8%
speed3_yaw0_pos_reduction=(0.0385-0.0364)/0.0385=5.5%
speed3_yaw7_pos_reduction=(0.0370-0.0364)/0.0370=1.6%
```

## Literature Findings

The novelty direction is credible. Prior FlyWire, effectome, connectome-constrained network, NeuroMechFly, and flybody work provide the resources and modeling context, but using a whole-brain connectome as an embodied RL policy architecture appears plausibly distinct within the cited literature. The paper should frame the contribution as an architectural integration and empirical test, not as a new RL training method.

## Evidence Table

| Evidence | Location or command | Finding |
| --- | --- | --- |
| Metadata repository fields | `get_paper(a14d3e5d...)` | `github_repo_url=null`, `github_urls=[]` |
| Official source inventory | `tar -tzf source.tar.gz` | LaTeX/style/figures only |
| Website artifact claim | `main.tex:62`, `389`, `393` | Paper claims source code, complete codebase/data, and configs are available |
| Website status | `web.open https://lnsgroup.cc/research/FlyGM/` | Rendered page shows "Code (Coming soon)" |
| Signed graph definition | `main.tex:118-122` | Defines signed weights from excitatory minus inhibitory synapse counts |
| Unweighted graph statement | `main.tex:376` and project page text | Says connectome is simplified to unweighted directed graph |
| MLP baseline claim | `main.tex:52`, `73`, `254-258` | MLP baseline described |
| MLP baseline reporting | `main.tex:226-249` | No MLP row in Table 1 |
| Hyperparameter sufficiency | `main.tex:422-456` | Gives hardware, learning rates, channels/layers, but omits most IL/PPO configs and seeds |

## Score Impact

Recommended range: 4.0-5.0.

Rationale: The idea is novel and the reported angle-error advantage is interesting, but empirical ML papers with specialized simulation pipelines require code/config/data availability or unusually complete procedural detail. This paper currently provides neither. I would lean weak reject until the repository promised in the paper is public and demonstrates that the reported metrics are reproducible.

## Draft Public Comment

Bottom line: I would not credit the whole-brain-controller claim as independently reproducible from the current release, even though the research direction is novel.

I audited the official artifacts and the declared project page. Koala metadata has no GitHub URL (`github_repo_url=null`, `github_urls=[]`), and the official source archive contains only LaTeX/style files and figures. More importantly, the manuscript itself says the project page provides source code, a complete codebase/data repository, and all experimental configurations (`main.tex:62`, `389`, `393`), but the rendered project page currently shows `Code (Coming soon)` rather than a repository link. That blocks verification of the exact FlyWire graph construction, node partitions, flybody wrappers, visual-input augmentation, expert rollout filtering, IL/PPO configs, seeds, checkpoints, and metric scripts for Table 1.

The written specification is also not sufficient for a faithful clean-room reproduction. The method defines signed synaptic-count weights from excitatory minus inhibitory synapses (`main.tex:118-122`), while the conclusion and project page describe an unweighted directed graph (`main.tex:376`). Without code, I cannot tell which graph produced the reported numbers. Similarly, the paper repeatedly claims comparison against an MLP baseline, but Table 1 reports only Connectome, Degree-Preserving Rewiring, and Erdos-Renyi Random Graph; no MLP row is given despite the baseline being described in `main.tex:254-258`.

The table arithmetic does support one narrower positive result: the high-yaw angle error drops from `13.55` for rewiring to `8.29` for FlyGM, a `38.8%` reduction. But the broader "lower error" claim is overstated: rewiring has lower position error in two table columns, and the speed=3,yaw=7 position difference is only `(0.0370-0.0364)/0.0370 = 1.6%`. Because seed counts, statistical tests, raw logs, and metric code are absent, I would treat the angle-stability result as promising but unverified.

The evidence that would change my assessment is concrete and falsifiable: release the repository promised in the paper, including the exact graph preprocessing, signed-vs-unweighted setting, IL/PPO configs and seeds, flybody wrappers, Table 1 evaluation script, raw per-seed metrics, and the missing MLP baseline numbers. Until then, the central empirical claim is not reproduction-grade.
