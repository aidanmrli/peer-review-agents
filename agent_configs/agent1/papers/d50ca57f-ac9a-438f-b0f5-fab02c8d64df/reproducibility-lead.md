# Reproducibility Lead Report

Paper ID: `d50ca57f-ac9a-438f-b0f5-fab02c8d64df`  
Title: "Transport Clustering: Solving Low-Rank Optimal Transport via Clustering"  
Role: Reproducibility Lead  
Date: 2026-04-24

## Task Scope

I coordinated the internal review team for a reproducibility-first assessment. The paper's decision-relevant claims were ranked as follows:

1. Transport Clustering reduces low-rank optimal transport to generalized K-means after a full-rank transport registration step and admits constant-factor approximation guarantees.
2. Transport Clustering empirically outperforms LOT, FRLC, and LatentOT/FactoredOT on synthetic, CIFAR-10, single-cell, Wasserstein-estimation, runtime, and ablation benchmarks.
3. The practical method using entropic Sinkhorn or HiRef registration inherits enough of the Monge-registration theory to support the paper's conclusions.
4. The novelty claim is accurate relative to prior low-rank OT and OT clustering literature.

The minimum reproduction target was set before seeing outcomes:

- At least one independently executable route to the central empirical comparisons, with enough code, data, seeds, and environment information to regenerate a table or figure within normal numerical tolerance.
- If execution was impossible, each independent reproducer would validate the smallest meaningful unit: artifact completeness, a table arithmetic check, a proof step, or an analytic benchmark target.
- For the theory, a proof reconstruction should find no unrepaired algebraic defect in the core hard-assignment reduction.

## Evidence Examined

Local artifacts:

- `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/paper.pdf`
- `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/source.tar.gz`
- `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex`
- `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/00README.json`
- rendered figure PDFs in `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/figures/`

Platform metadata:

- Koala reports `github_repo_url: null` and `github_urls: []`.
- The paper is currently `in_review`.
- Existing comments are one bibliography-format check and one positive general review.

Key commands:

```bash
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.csv' -o -name '*.npy' -o -name '*.npz' -o -name 'requirements*' -o -name 'environment*' \) | sort
rg -n "github|code|available|implementation|environment|requirements|seed|random|hardware|GPU|CPU|ott-jax|HiRef|scanpy|scikit|Sinkhorn|subsample|slightly|epsilon|iterations|runtime|failed|fails" papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex
sed -n '1,120p' papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/00README.json
```

The executable/data search returned only `artifacts/00README.json`, which describes the LaTeX build. It found no runnable implementation or machine-readable result artifacts.

## Role Findings

Independent Reproducer A:

- Attempted to reproduce the central empirical superiority claim from official artifacts.
- Found exact empirical reproduction blocked because the bundle contains LaTeX, figures, and tables only.
- Independently reproduced only the fragmented-hypercube target `W_2^2 = 8` from the analytic formula.
- Checked selected table arithmetic, including entropy-sensitivity ratios, but these checks only validate reported numbers against other reported numbers.

Independent Reproducer B:

- Used a separate source/table/proof reconstruction route.
- Again found empirical reconstruction blocked by missing code, raw logs, configs, and repository.
- Reconstructed the high-level theorem route partially, but identified a hard-assignment normalization error in `main.tex:1217-1231`.
- Flagged internal protocol inconsistencies: 315 versus 320 synthetic-instance accounting, 2M-8G noise-level mismatch, and duplicate `fig:twomoons` labels.

Implementation Auditor:

- Found no GitHub repository and no code paths to inspect.
- Confirmed the artifact bundle has only paper source, bibliography/style files, and static figure PDFs.
- Marked synthetic, CIFAR-10, single-cell, Wasserstein-estimation, runtime, and ablation claims as not independently verifiable from the submitted artifacts.
- Identified missing package versions, exact seeds, HiRef/GKMS commands, baseline invocation, preprocessing scripts, raw result logs, and hardware/timing details.

Correctness Specialist:

- Found the main guarantee is for hard, uniform, equal-size, Monge/permutation low-rank OT, while the paper repeatedly advertises standard low-rank Kantorovich OT.
- Found the experiments use approximate Sinkhorn or HiRef registration rather than the exact permutation registration required by the proof.
- Identified an ambiguous chained theorem statement, under-specified `gamma`/`rho`, a scaling error in the hard-assignment equivalence, a dimension error in the Kantorovich-registration feasibility line, and gaps between the GKMS descent proposition and the fixed-step implementation.
- Did not find a single fatal contradiction that invalidates the intended hard-assignment theorem after repair.

Literature Specialist:

- Found the Monge-registered reduction appears genuinely novel relative to Forrow, Scetbon, LatentOT, and FRLC.
- Found the paper under-discusses OT co-clustering and OT clustering work, especially Laclau et al. 2017, Genevay et al. 2019, and Wasserstein K-means.
- Concluded the novelty is real but the framing should more sharply distinguish inherited K-means/LR-OT connections from the new registration reduction.

## Reproduction Outcome

Central empirical claim: weak reproducibility.

Neither Independent Reproducer A nor Independent Reproducer B could reproduce the central experimental tables or figures. The Implementation Auditor independently confirmed the same blocker. The official artifacts do not provide the implementation, commands, raw results, exact seeds, data processing scripts, environment, or repository needed to rerun the claims.

Central theory claim: partial reproducibility.

The high-level hard-assignment proof route is inspectable and partly reconstructible. However, the current source has proof-writing and scope defects, including the normalization error at `main.tex:1217-1231` and the theory-practice mismatch between exact Monge registration and approximate Sinkhorn/HiRef registration.

Small analytic claim: reproduced.

The fragmented-hypercube benchmark target `W_2^2 = 8` is independently derivable from the transformation formula. This is useful but not sufficient evidence for the reported method comparisons.

## Remaining Uncertainty

The empirical results may be correct, but they are asserted rather than reproducible from the provided artifacts. The missing code and raw outputs matter because the reported gains depend on solver initialization, full-rank registration quality, low-rank GKMS implementation, baseline tuning, large-scale preprocessing, and metric computation.

The theoretical idea is promising, and the literature specialist found a real novelty signal. The remaining uncertainty is whether the current proofs can be repaired cleanly and whether the practical implementation is faithful to the mathematical object being certified.

## Decision Impact

For a reproducibility-first review, this paper should be marked down materially. The Monge-registered reduction is interesting enough to prevent a clear reject based on novelty alone, but the main empirical acceptance case is weakly reproducible and the theory is narrower than the abstract/introduction suggest. My current score impact is weak reject to borderline, roughly `4.0-5.0`, pending author release of code/raw results and repair of the theory-scope issues.
