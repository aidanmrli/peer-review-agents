# Implementation Auditor Report

Paper: d50ca57f-ac9a-438f-b0f5-fab02c8d64df, "Transport Clustering: Solving Low-Rank Optimal Transport via Clustering"

Role: Implementation Auditor

Date: 2026-04-24

## Bottom Line

Severity for acceptance decision: **High**.

The paper's empirical claims are not independently reproducible from the submitted artifacts. Koala metadata provides no GitHub URL, and the local artifact bundle contains only LaTeX/PDF source, bibliography/style files, and rendered figure PDFs. There is no implementation of Transport Clustering, no baseline code, no experiment scripts, no environment specification, no data preprocessing code, no raw result tables/logs, and no command-level reproduction instructions. The algorithm is specified at a mathematical/prose level well enough for a fresh implementation attempt, but not well enough to reproduce the reported synthetic, CIFAR-10, single-cell, runtime, Wasserstein-estimation, or ablation numbers.

## Scope and Commands Run

I inspected only the paper PDF/source artifacts under `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts`, as assigned.

Commands used:

```bash
curl -fsSL https://koala.science/skill.md
sed -n '1,240p' skills/implementation-auditor.md
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df -maxdepth 3 -type f -print | sort
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts -maxdepth 3 -type d -print | sort
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts -maxdepth 2 -type f -printf '%p %s bytes\n' | sort
tar -tzf papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/source.tar.gz
rg -n -i "github|repository|source code|code available|available at|implementation is available|artifact|reproduce" papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/references.bib
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.sh' -o -name '*.yaml' -o -name '*.yml' -o -name '*.json' -o -name '*.csv' -o -name '*.npy' -o -name '*.npz' -o -name '*.h5ad' -o -name '*.txt' -o -name '*.md' -o -name 'requirements*' -o -name 'environment*' -o -name 'Dockerfile' -o -name 'Makefile' \) -print | sort
rg -n "(CIFAR|single|cell|synthetic|Gaussian|two moons|SBM|experiment|implementation|GitHub|github|code|repository|data|dataset|runtime|Table|Figure|Algorithm|Sinkhorn|LOT|rank|epsilon|training|hyper|seed|supplement|appendix)" papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex
```

The broad executable/data/config search returned only `artifacts/00README.json`; no `.py`, `.ipynb`, shell scripts, environment files, CSV/NPY/NPZ/H5AD data, or Makefile were present. The `rg -n -i` code-availability query returned no matches.

## Artifact Inventory

The artifact directory contains 21 files:

- `paper.pdf`
- `source.tar.gz`
- `00README.json`
- `main.tex`
- `references.bib`
- ICML/LaTeX style files: `icml2026.sty`, `icml2026.bst`, `algorithm.sty`, `algorithmic.sty`, `fancyhdr.sty`
- Rendered figure PDFs: `TC_Convergence_Varying_N_small.pdf`, `Transport_Conjugation_CostMat.pdf`, `correspondence_clustering.pdf`, `epsilon_sensitivity.pdf`, `results1.pdf`, `runtimes.pdf`, `sbm_clustering.pdf`, `sbm_cost_ratio.pdf`, `shifted_gaussians_clustering.pdf`, `shifted_gaussians_cost_ratio.pdf`, `two_moons_cost_ratio.pdf`

`00README.json` only describes the LaTeX build: top-level `main.tex`, compiler `pdflatex`, TexLive 2025. The `source.tar.gz` listing matches the same LaTeX/figure/style contents and does not contain code or data.

## Code Paths Inspected

There were no code paths to inspect. No GitHub repository was provided in Koala metadata, and the local source does not contain a code-availability statement or repository URL. Consequently, I could not audit:

- the Transport Clustering implementation;
- the JAX `GKMS` implementation;
- the low-rank factorized `GKMS` variant used for large datasets;
- the HiRef integration;
- the `ott-jax` Sinkhorn calls;
- the `LOT`, `FRLC`, `LIN`, and FactoredOT baseline wrappers;
- the SDP initialization for SBM;
- CIFAR-10 ResNet/PCA preprocessing;
- single-cell `scanpy` preprocessing and subsampling;
- metric computation for OT cost, AMI, ARI, CTA, runtime, and Wasserstein estimation error.

## Paper Claims and Artifact Support

### Main empirical claim

Claim tested: Transport Clustering empirically outperforms existing low-rank OT solvers on synthetic benchmarks and large-scale high-dimensional datasets.

Paper locations:

- Abstract states empirical superiority on synthetic and large-scale datasets, `main.tex` lines 197-201.
- Synthetic results claim consistent best low-rank OT cost across synthetic datasets, lines 1076-1112.
- CIFAR-10 table and text report TC cost 231.200, AMI/ARI 0.478/0.358 and 0.476/0.356, CTA 0.412, lines 1114-1143.
- Single-cell results claim TC lower OT cost, higher AMI/ARI/CTA across six timepoint pairs and report runtimes, lines 1145-1146 and 2702-2748.
- Wasserstein estimation claim TC gives the most accurate estimate across sample sizes, lines 1148-1166 and 2612-2628.
- Ablation claims cover entropy sensitivity, initialization, and Kantorovich registration, lines 2634-2700.

Artifact support: **Not sufficient**. Only final plotted PDFs and LaTeX table values are available. No raw numerical logs, scripts, seeds, saved couplings, data splits, generated synthetic datasets, or baseline outputs are present.

Severity: **High**. These are central acceptance claims and cannot be independently verified.

### Algorithm specification

The paper specifies the high-level Transport Clustering procedure in Algorithm 1: compute a full-rank plan, register the cost matrix, solve generalized K-means, and output `(Q, P^T Q)` (`main.tex` lines 593-607). It also describes `GKMS` as an exponentiated-gradient/mirror-descent update with a row-marginal Sinkhorn projection, lines 968-984 and 2019-2125. Experimental implementation details state synthetic Sinkhorn used `ott-jax` with entropy `1e-5` and 10,000 iterations; real data used HiRef and a low-rank factorized `GKMS`; `GKMS` used JAX, step size `gamma_k = 2`, 250 iterations, scikit-learn K-means initialization, and a random centering mixture with `lambda = 1/2`, lines 2478-2508.

This is enough to guide an independent reimplementation of the idea, but not enough to reproduce the paper's exact results. Missing details include exact package versions, random seeds beyond the synthetic seed set, initialization options, K-means parameters, stopping/rounding rules, gradient implementation, numerical dtype, hardware, HiRef settings, low-rank factorization details, baseline hyperparameters, and failure criteria for methods reported as unavailable.

Severity: **Medium to High**. The method is conceptually implementable, but the reported numbers are not reproducible without author code or substantially more procedural detail.

## Reproducibility by Experiment Family

### Synthetic experiments

Paper details:

- Datasets: 2-Moons to 8-Gaussians, shifted Gaussians, SBM, each with `n=m=5000`, lines 1076-1078 and 2513-2527.
- Ranks and seeds: ranks `K in {50,75,...,250}` and `K in {10,...,100}` with seeds `{1,2,3,4,5}`, lines 2520-2524.
- Synthetic data definitions are given for 2M-8G, shifted Gaussians, and SBM, lines 2529-2580.
- Claimed results include 315 synthetic instances and relative cost plots, lines 949-955.

Missing artifacts:

- no synthetic data-generation scripts;
- no exact random seed wiring across NumPy/JAX/PyTorch/scikit-learn/graph generation;
- no code for cost construction at scale;
- no baseline command lines or hyperparameters for `LOT`, `FRLC`, `LIN`;
- no raw per-seed/per-rank results behind `results1.pdf`, `runtimes.pdf`, `sbm_cost_ratio.pdf`, `shifted_gaussians_cost_ratio.pdf`, or `two_moons_cost_ratio.pdf`;
- no saved couplings or cluster assignments to recompute ARI/AMI.

Cannot independently verify:

- TC's average 23% improvement over LOT on shifted Gaussians;
- TC's average 4% improvement over LOT on SBM;
- TC's relative cost dominance across 315 synthetic instances;
- runtime comparisons versus rank;
- co-clustering ARI/AMI values for SG and SBM;
- whether baselines were tuned comparably or run with faithful implementations.

Severity: **High**.

### CIFAR-10 experiment

Paper details:

- CIFAR-10 has 60,000 images; the paper embeds with `resnet18-f37072fd.pth`, reduces to PCA dimension 50, makes a stratified 50/50 split, uses fixed seeds, rank `K=10`, HiRef for registration, and mirror descent for generalized K-means, lines 1141-1143 and 2582-2595.
- Reported values include TC OT cost 231.200, AMI/ARI 0.478/0.358 and 0.476/0.356, CTA 0.412, lines 1123-1126.

Missing artifacts:

- no CIFAR download/preprocessing script;
- no ResNet feature extraction code or torchvision/model version;
- no PCA fitting code, normalization convention, or saved PCA features;
- no split file or exact seed value;
- no HiRef command/config;
- no baseline configurations;
- no metric implementation for OT cost, AMI, ARI, or CTA;
- no raw result file for the reported table.

Cannot independently verify:

- the reported TC/LOT/FRLC CIFAR cost ordering;
- the AMI/ARI and CTA values;
- the claim that the split preserves matched class distributions;
- whether ResNet/PCA preprocessing follows the cited protocol exactly;
- whether the coupling used for CTA was formed and normalized consistently across methods.

Severity: **High**.

### Single-cell mouse embryogenesis experiment

Paper details:

- The paper aligns first replicate across E8.5 through E10.0 for six adjacent timepoint pairs from a 12.4M-nuclei sci-RNA-seq3 dataset, lines 1145-1146 and 2596-2598.
- It uses `scanpy` to read H5AD files, applies `sc.pp.normalize_total`, `sc.pp.log1p`, `sc.tl.pca` with randomized SVD to 50 PCs, performs slight subsampling to make `n` divisible for HiRef, balances cell-type proportions using `cell_id` from `df_cell.csv`, sets rank to the minimum cell-type count, and reports OT cost, AMI/ARI, CTA, and runtime, lines 2598-2599 and 2702-2748.

Missing artifacts:

- no H5AD files, metadata files, accession/download instructions, or checksums;
- no script identifying exact first-replicate sample IDs;
- no subsampling rule, divisibility target, or random seed;
- no code for class balancing;
- no `scanpy` version or PCA solver parameters beyond `"randomized"`;
- no HiRef and low-rank `GKMS` configs;
- no baseline configs or logs proving `LOT` failed beyond 45,360 cells;
- no raw result logs for OT cost, AMI, ARI, CTA, or runtime.

Cannot independently verify:

- all six single-cell alignment table rows;
- the claim that TC improves lower OT cost/higher AMI/ARI/CTA on all pairs;
- the reported runtime scaling;
- the failure boundary for `LOT`;
- the class balancing and cell-type annotation pipeline.

Severity: **High**.

### Wasserstein estimation experiment

Paper details:

- The paper uses the fragmented hypercube benchmark of Forrow et al. with target `W_2^2 = 8`, dimension 30, rank 10, averaged over 10 runs, lines 1148-1166 and 2600-2628.

Missing artifacts:

- no benchmark generation script;
- no exact sample-size loop implementation;
- no seed list for the 10 runs;
- no baseline implementations/configs for full-rank OT, K-Means, FactoredOT, FRLC, and TC;
- no raw per-run errors or standard deviations.

Cannot independently verify:

- the table of estimation errors;
- the claim that TC achieves the most accurate estimate across sample sizes;
- variance or statistical stability of the reported averages.

Severity: **High**.

### Ablations

Paper details:

- Entropy sensitivity varies `epsilon_i=10^i` for `i in {-5,...,1}`, lines 2656-2681.
- Kantorovich registration fixes `n=1024` and varies `m in {1024,512,256,128,64}`, lines 2638-2654.
- Initialization ablation reports TC, FRLC random, and FRLC TC-initialized costs on planted Gaussians, lines 2683-2700.

Missing artifacts:

- no ablation scripts;
- no seed lists;
- no low-rank solver configs;
- no saved raw outputs;
- no validation of the plotted/table values.

Cannot independently verify:

- entropy sensitivity cost values;
- the claim that low entropy is responsible for major improvement;
- generalization to Kantorovich registration with asymmetric sample sizes;
- the claim that TC initialization accounts for most practical improvement.

Severity: **Medium to High**.

## Paper-to-Code Matches

None could be established, because no code was released in the artifact bundle and no repository URL was present. The only possible comparison is paper-to-prose: the artifact's `main.tex` contains the mathematical algorithm and experimental descriptions that appear in `paper.pdf`.

## Paper-to-Code Discrepancies and Risks

Because there is no code, every implementation-sensitive claim remains unaudited. Specific risks:

- **Baseline fairness risk:** `LOT`, `FRLC`, `LIN`, and FactoredOT performance depends heavily on initialization, regularization, iteration counts, stopping criteria, and implementation. These are not auditable.
- **HiRef dependency risk:** Real-data and CIFAR claims rely on HiRef, but the paper provides no commands or parameters for the HiRef registration step.
- **Metric risk:** OT cost, AMI, ARI, CTA, and Wasserstein estimation error can vary with normalization and coupling construction. Metric code is absent.
- **Data provenance risk:** CIFAR preprocessing and single-cell data selection/subsampling are not reproducible from the artifact bundle.
- **Runtime risk:** Reported runtimes cannot be interpreted without hardware, dtype, JIT compilation policy, warmup handling, or timing code.
- **Raw-results risk:** Rendered PDFs and LaTeX tables are not audit trails; they do not allow checking per-seed variability, failed runs, or plotting/data-processing mistakes.

## Dependency and Environment Audit

Dependencies named in the paper include `ott-jax`, `JAX`, `scikit-learn`, `torchdyn.datasets`, `scanpy`, ResNet weight `resnet18-f37072fd.pth`, and HiRef. No dependency versions, lockfile, Dockerfile, conda/pip environment, hardware details, CUDA/JAX backend details, or package-specific settings are provided.

Severity: **High** for exact reproduction, **Medium** for independent reimplementation.

## Algorithm Implementability

Transport Clustering itself is specified at a high mathematical level:

1. compute a full-rank OT plan;
2. register the cost matrix;
3. solve generalized K-means for `Q`;
4. form the second low-rank factor from the registration.

The paper gives enough equations for a competent reviewer to attempt a clean-room implementation of the core method on small synthetic data. However, exact reproduction of the submitted results is blocked by missing implementation details:

- exact gradient implementation for `GKMS` as used in code;
- final rounding or hard-assignment procedure after dense mirror-descent iterates;
- lower-bound/clipping behavior for `Q^T 1_n`;
- low-rank factorized cost path `C = AB^T`;
- handling of rectangular/asymmetric Kantorovich registration;
- baseline initialization parity;
- numerical tolerances and stopping logic;
- JIT/warmup and memory strategies for large datasets.

Severity: **Medium** for the algorithm description, **High** for reproducing empirical claims.

## Claims Not Independently Verifiable Because of Missing Artifacts

The following claims cannot be verified from the submitted artifacts:

- TC outperforms existing low-rank OT solvers on synthetic benchmarks and large-scale high-dimensional datasets.
- TC is consistently best in low-rank OT cost across synthetic tasks.
- TC improves shifted-Gaussian low-rank OT cost by an average 23% over LOT.
- TC improves SBM low-rank OT cost by an average 4% over LOT.
- TC has the reported synthetic ARI/AMI values.
- CIFAR-10 TC achieves OT cost 231.200, AMI/ARI 0.478/0.358 and 0.476/0.356, and CTA 0.412.
- TC beats LOT and FRLC on CIFAR-10 under matched preprocessing and fair baseline settings.
- Single-cell TC obtains the six reported OT cost, AMI/ARI, CTA, and runtime rows.
- LOT fails beyond the stated single-cell scale while TC and FRLC scale to all pairs.
- TC achieves the best fragmented-hypercube Wasserstein estimation errors across most sample sizes.
- Entropy regularization, TC initialization, and Kantorovich registration ablation conclusions.
- Runtime comparisons in `runtimes.pdf`.

## Final Assessment

The implementation/artifact package is inadequate for a reproducibility-first ICML review. The paper's theoretical algorithm is described in enough detail to be studied and partially reimplemented, but the empirical acceptance case relies on large, implementation-sensitive experiments whose code, configurations, data splits, dependency environment, baseline settings, and raw outputs are absent. The absence of a GitHub repository is therefore not cosmetic; it directly prevents verification of the paper's central empirical claims.
