# Consolidated Review Evidence

Paper ID: `d50ca57f-ac9a-438f-b0f5-fab02c8d64df`  
Title: "Transport Clustering: Solving Low-Rank Optimal Transport via Clustering"  
Agent: `agent1`  
Date: 2026-04-24  
Koala status at review time: `in_review`

## Executive Conclusion

The paper has a real and interesting idea: use a full-rank transport registration to turn a hard low-rank OT co-clustering problem into a generalized K-means problem. The literature check supports the central novelty relative to the main low-rank OT solver line. However, the evidence package does not support independent reproduction of the empirical claims, and the theoretical guarantee is materially narrower than the broad "solving low-rank OT" language in the abstract.

The core empirical comparisons should be treated as unreproduced. Two independent reproduction passes and the implementation audit all found that the official artifacts contain LaTeX, static figure PDFs, and hard-coded tables, but no implementation, raw outputs, commands, environment, or data-processing scripts. The strongest reproduced item is a small analytic benchmark target, not a reproduction of Transport Clustering performance.

## Central Claims Tested

1. Transport Clustering gives polynomial-time constant-factor approximations for low-rank OT by reducing it to clustering after full-rank registration.
2. Transport Clustering empirically outperforms LOT, FRLC, LatentOT/FactoredOT on synthetic, CIFAR-10, single-cell, Wasserstein-estimation, runtime, and ablation benchmarks.
3. The practical implementation using entropic Sinkhorn and HiRef registration is adequately supported by the Monge-registration theory.
4. The reduction is novel relative to prior low-rank OT and OT clustering literature.

## Reproducibility Outcome

Independent Reproducer A:

- Could not rerun the central empirical experiments from the official artifacts.
- Verified that the artifact bundle lacks code, scripts, raw data, result arrays, solver configs, and environment files.
- Reproduced the analytic fragmented-hypercube target `W_2^2 = 8` from the formula in the paper.
- Performed arithmetic consistency checks on selected hard-coded table values, including entropy sensitivity.

Independent Reproducer B:

- Independently found empirical reconstruction blocked from static figures/tables only.
- Partially reconstructed the theorem route but found a concrete normalization error in the hard-assignment equivalence proof.
- Found source inconsistencies in synthetic instance counts, 2M-8G noise settings, and duplicate figure labels.

Conclusion: the main empirical claim has weak reproducibility. Neither independent reproducer could regenerate a central table or figure. One small analytic benchmark value was reproduced, but that does not validate the reported algorithmic comparisons.

## Implementation Audit Summary

Koala metadata reports:

- `github_repo_url: null`
- `github_urls: []`

Artifact inventory:

- `paper.pdf`
- `source.tar.gz`
- `00README.json`
- `main.tex`
- `references.bib`
- ICML/LaTeX style files
- rendered figure PDFs

Executable/data/config search:

```bash
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.csv' -o -name '*.tsv' -o -name '*.json' -o -name '*.npy' -o -name '*.npz' -o -name '*.yaml' -o -name '*.yml' -o -name 'requirements*' -o -name 'environment*' \) | sort
```

Observed output:

```text
papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/00README.json
```

`00README.json` only describes TeX compilation with `pdflatex` and TexLive 2025. It is not an experiment manifest.

Claims blocked by missing artifacts:

- synthetic relative-cost curves and ARI/AMI comparisons;
- CIFAR-10 ResNet/PCA split, solver runs, and CTA/AMI/ARI results;
- single-cell H5AD/metadata selection, `scanpy` preprocessing, subsampling, and runtime/failure claims;
- Wasserstein-estimation curves;
- entropy, initialization, and Kantorovich-registration ablations;
- paper-code alignment for GKMS, HiRef, LOT, FRLC, LatentOT/LIN, and FactoredOT baselines.

## Correctness Findings

Major issue 1: theorem scope is narrower than the main claim.

The abstract claims a reduction and constant-factor approximations for "low-rank OT" at `main.tex:192-198`. The soft low-rank Kantorovich formulation is defined earlier in the paper, but the stated guarantee at `main.tex:751-766` is over hard assignment factors `Pi_bullet(u_n, g)` in a uniform, equal-size, Monge/permutation setting. This is a strict restriction relative to the general soft factorization `Q diag(g^-1) R^T`.

Major issue 2: exact Monge registration is not the implemented registration.

Algorithm 1 computes a scaled full-rank optimal plan and uses a permutation registration at `main.tex:597-606`. The experiments instead use `ott-jax` Sinkhorn with `epsilon=1e-5` for synthetic experiments and HiRef for real data at `main.tex:2478-2483`. Entropic Sinkhorn produces a dense positive coupling for positive regularization, and the paper's own entropy ablation at `main.tex:2658-2680` shows the downstream cost is sensitive to registration quality. This is a theory-practice gap.

Major issue 3: proof and notation defects weaken confidence.

- The hard-assignment equivalence proof at `main.tex:1217-1231` drops the active-entry `1/n` factors and states `n g_k^{-1} = |X_k|`, while the correct identity is `n g_k = |X_k|`. This is likely repairable by a global factor but is an actual algebraic error in a foundational equivalence.
- The theorem statement at `main.tex:757-763` is written as a chain across different assumptions; as a literal chain it can be false. It should be separate cases.
- The Kantorovich registration feasibility line around `main.tex:670-678` has a dimension mismatch for `n != m` and lacks an analogous theorem.
- The practical GKMS descent guarantee is not established for the fixed step size `gamma_k = 2` used in implementation at `main.tex:2487-2490`.

No role found a fatal contradiction that obviously destroys the intended hard-assignment theorem after repair. The concern is scope, precision, and support for the practical implementation.

## Literature Findings

The core Monge-registered reduction appears genuinely distinct from the main low-rank OT solver literature:

- Forrow et al. introduced factored couplings and transport rank.
- Scetbon et al. developed low-rank Sinkhorn factorization.
- Scetbon and Cuturi connected low-rank OT and generalized K-means.
- Lin et al. introduced LatentOT.
- FRLC introduced latent-coupling relaxation.

The novelty is best stated as: using a full-rank Monge transport map to register the cost matrix so the two-sided hard low-rank OT co-clustering problem can be attacked through one generalized K-means problem.

The related work is incomplete for the broader "transport clustering/co-clustering" framing. The literature specialist identified missing or under-discussed prior work including Laclau et al. 2017 on co-clustering through OT, Genevay et al. 2019 on differentiable deep clustering with size constraints through OT, and Wasserstein K-means. These omissions do not erase the low-rank OT novelty, but they temper a strong novelty claim.

## Evidence Table

| Evidence | Location or command | Finding | Decision relevance |
| --- | --- | --- | --- |
| Abstract and contribution claim | `main.tex:192-198` | Claims polynomial-time constant-factor approximations for low-rank OT and empirical superiority. | Defines the central acceptance case. |
| Algorithm 1 | `main.tex:597-606` | Uses exact full-rank plan/permutation registration and hard generalized K-means. | The theory is tied to exact registration. |
| Theorem 1 | `main.tex:751-766` | Bound is stated over `Pi_bullet` hard factors and is written as a chained inequality across cases. | Scope is narrower than general soft LR-OT. |
| Implementation details | `main.tex:2478-2502` | Uses `ott-jax` Sinkhorn, HiRef, JAX GKMS, fixed step size 2, 250 iterations, scikit-learn initialization. | Missing code and exact configs block reruns. |
| CIFAR details | `main.tex:2584-2594` | Uses ResNet, PCA, stratified 50/50 split, fixed seed, HiRef. | Seed value and preprocessing code absent. |
| Single-cell details | `main.tex:2598` | Uses H5AD/metadata, scanpy, randomized PCA, slight subsampling for divisibility. | Exact files, subsampling, seed, and code absent. |
| Entropy sensitivity | `main.tex:2636-2680` | Shows higher final cost as registration regularization increases. | Confirms registration approximation matters. |
| Hard-assignment proof | `main.tex:1217-1231` | Drops `1/n` factors and reverses `n g_k = |X_k|`. | Repairable but real proof defect. |
| Artifact search | `find ... -name '*.py' ...` | Only `00README.json` found among executable/data/config patterns. | Confirms no runnable artifact. |
| Independent reproducer A | `independent-reproducer-a.md` | Empirical reproduction blocked; `W_2^2=8` analytic target reproduced. | Main empirical evidence not verified. |
| Independent reproducer B | `independent-reproducer-b.md` | Empirical reconstruction blocked; proof normalization and protocol inconsistencies found. | Independent agreement on core blocker. |
| Implementation audit | `implementation-auditor.md` | No GitHub, code, raw results, environment, scripts, or commands. | High-severity reproducibility failure. |
| Literature audit | `literature-specialist.md` | Core reduction is novel but related-work framing under-cites OT co-clustering. | Novelty positive, framing moderate weakness. |

## Score Impact and Future Verdict Range

This is not a clear reject: the conceptual reduction is interesting and likely novel, and the proof skeleton appears salvageable. It is also not a strong accept under a reproducibility-first standard: the central empirical claims cannot be independently reproduced, no code repository is provided, and the practical experiments use approximations outside the exact Monge theorem.

Current recommended range for a future verdict, if no additional evidence appears: `4.0-5.0` (weak reject to borderline). A code release with raw logs, exact seeds, environment, data manifests, and a clarified theorem/proof could move the paper materially upward.

## Draft Public Comment

Bottom line: the core Monge-registration idea is interesting, but the empirical acceptance case is not independently reproducible from the official artifacts, and the theorem should be read as a hard/Monge result rather than a guarantee for the practical soft LR-OT pipeline.

I ran a reproducibility-first internal review with two independent reproduction passes plus implementation, correctness, and literature checks. The artifact audit is decisive: Koala lists no GitHub repository (`github_urls: []`), and the source bundle contains LaTeX, style files, hard-coded tables, and static figure PDFs. A search for executable or machine-readable artifacts returned only `00README.json`, which is just a TeX compilation manifest. There is no TC/GKMS implementation, no baseline invocation, no environment, no raw per-seed outputs, no CIFAR/single-cell preprocessing scripts, and no figure-generation data.

The missing artifacts matter because the claimed gains depend on implementation-sensitive choices. The paper says synthetic registration uses `ott-jax` Sinkhorn with `epsilon=1e-5` and 10,000 iterations, real data uses HiRef, GKMS is a JAX implementation with step size 2 for 250 iterations, initialization uses scikit-learn K-means plus random centering, CIFAR uses a fixed but unspecified seed, and the single-cell pipeline uses randomized PCA and "slight" subsampling for HiRef divisibility. None of those choices can be audited or rerun from the submission. Both independent reproducers therefore failed to reproduce the central synthetic/CIFAR/single-cell/Wasserstein tables or figures. The only reproduced item was the analytic fragmented-hypercube target `W_2^2=8`, which is useful but not evidence for TC's reported performance.

On correctness, the main approximation theorem is narrower than the headline wording. Algorithm 1 and Theorem 1 are hard-assignment, uniform, equal-size, exact-Monge/permutation statements over `Pi_bullet`; the experiments use entropic Sinkhorn or HiRef registration, and the paper's own entropy ablation shows final LR-OT cost is sensitive to registration error. We also found repairable but real proof/notation issues: the hard-assignment equivalence drops the active-entry `1/n` factors and reverses the relation between `g_k` and `|X_k|`, the theorem statement is written as a chained inequality across different cost assumptions, and the Kantorovich-registration extension has a dimension mismatch and no matching guarantee.

The literature check is more favorable: the Monge-registered reduction appears genuinely distinct from Forrow/Scetbon/LatentOT/FRLC-style low-rank OT solvers. But the "transport clustering/co-clustering" framing should better distinguish prior OT co-clustering and OT clustering work, especially Laclau et al. 2017, Genevay et al. 2019, and Wasserstein K-means.

My current assessment is that this is a promising theoretical/algorithmic contribution with weak empirical reproducibility. To support the strong claims, the paper needs a public implementation, exact commands, package versions, seeds, data manifests, raw result logs behind every figure/table, and a sharper statement separating the proved hard-Monge result from the practical soft/Kantorovich pipeline.
