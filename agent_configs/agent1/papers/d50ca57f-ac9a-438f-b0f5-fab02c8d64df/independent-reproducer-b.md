# Independent Reproducer B Report

Paper: `d50ca57f-ac9a-438f-b0f5-fab02c8d64df`, "Transport Clustering: Solving Low-Rank Optimal Transport via Clustering"

Role: adversarial alternate-route reproduction from LaTeX source, algorithms, figures, and tables. I did not use Reproducer A's notes; no Reproducer A report was present in this paper directory during this pass.

## Claim Attempted

I attempted to independently reconstruct two central claim classes:

1. The theoretical reduction claim: Transport Clustering solves hard low-rank OT by Monge registration plus generalized K-means and has constant-factor approximation guarantees, especially `(1 + gamma)` for negative-type metrics and `(1 + gamma + sqrt(2 gamma))` for kernel costs.
2. The empirical table/figure claim: Transport Clustering obtains lower low-rank OT cost and competitive or better co-clustering metrics than LOT, FRLC, and LatentOT/LIN on synthetic, CIFAR-10, single-cell, and Wasserstein-estimation experiments.

## Independent Route Used

I used the artifact source rather than rendered PDF text extraction:

- Koala metadata via `get_paper` for repository and artifact availability.
- Local source bundle:
  - `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex`
  - `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/source.tar.gz`
  - `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/figures/*.pdf`
  - `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/00README.json`

I did not inspect or rely on OpenReview reviews, citation counts, later discussion, or post-release commentary.

## Commands and Observations

Platform and artifact availability:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,240p' skills/independent-reproducer-b.md
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df -maxdepth 2 -type f -print | sort
tar -tzf papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/source.tar.gz | sort | sed -n '1,240p'
```

Observed: Koala reports `github_urls: []` and `github_repo_url: null`. The local bundle has the PDF, `main.tex`, bibliography, style files, and static PDF figures, but no implementation scripts, notebooks, raw logs, result CSVs, configuration files, environment files, or data-processing code.

Source inspection:

```bash
rg -n "Table|Figure|Theorem|Proposition|Corollary|Lemma|Algorithm|transport|rank|cluster|guarantee|approx|experiment|baseline|implementation|github|code|seed|hyper" papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex
nl -ba papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex | sed -n '580,650p'
nl -ba papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex | sed -n '720,765p'
nl -ba papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex | sed -n '1180,1245p'
nl -ba papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex | sed -n '2476,2508p'
```

Observed source locations:

- Algorithm 1 is in `main.tex:593-606`: compute a full-rank plan, register `C P_sigma^T`, solve generalized K-means, output `(Q, P_sigma^T Q)`.
- Theorem 1 is in `main.tex:724-765`: it states the negative-type, kernel-cost, and general-metric approximation factors.
- The initialization theorem is in `main.tex:917-946`.
- Implementation details are in `main.tex:2478-2508`: synthetic Monge map via `ott-jax` Sinkhorn with epsilon `1e-5` and 10,000 iterations; real-data Monge map via HiRef; GKMS in JAX with step size 2 for 250 iterations; scikit-learn K-means initialization; a random centering step.

Figure and table inspection:

```bash
ls -la papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/figures
file papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/figures/*.pdf
rg -n -F '\begin{table}' papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex
rg -n -F '\caption{' papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex
rg -n -F '\label{fig:' papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex
```

Observed: all figures are static PDFs. The tables are hard-coded in LaTeX. I found no raw data behind `results1.pdf`, `two_moons_cost_ratio.pdf`, `shifted_gaussians_cost_ratio.pdf`, `sbm_cost_ratio.pdf`, `runtimes.pdf`, `TC_Convergence_Varying_N_small.pdf`, or the table values.

Manual normalization check for the hard-assignment proof:

```bash
python3 - <<'PY'
n=4
X_clusters=[[0,1],[2,3]]
Y_clusters=[[1,3],[0,2]]
C=[[1+i*n+j for j in range(n)] for i in range(n)]
Q=[[0.0]*2 for _ in range(n)]
R=[[0.0]*2 for _ in range(n)]
for k,cl in enumerate(X_clusters):
    for i in cl: Q[i][k]=1/n
for k,cl in enumerate(Y_clusters):
    for j in cl: R[j][k]=1/n
g=[sum(Q[i][k] for i in range(n)) for k in range(2)]
obj=0.0
for i in range(n):
    for j in range(n):
        pij=sum(Q[i][k]*(1/g[k])*R[j][k] for k in range(2))
        obj += C[i][j]*pij
part=0.0
for k in range(2):
    s=sum(C[i][j] for i in X_clusters[k] for j in Y_clusters[k])
    part += s/len(X_clusters[k])
print('g =', g)
print('assignment_objective =', obj)
print('partition_objective =', part)
print('n * assignment_objective =', n*obj)
print('n*g =', [n*x for x in g], 'not n/g =', [n/x for x in g])
PY
```

Observed output:

```text
g = [0.5, 0.5]
assignment_objective = 8.5
partition_objective = 34.0
n * assignment_objective = 34.0
n*g = [2.0, 2.0] not n/g = [8.0, 8.0]
```

## Reconstruction Result

### Theory: Partial Match, With a Source-Level Normalization Error

The high-level theorem structure is reconstructible from the source. The proof route is coherent at a broad level: reduce hard low-rank OT to the partition form, fix the Monge permutation, compare feasible registered partitions to the low-rank optimum, and use metric/CND/kernel inequalities to obtain the stated factors.

However, the assignment-to-partition equivalence proof has a normalization error in the written source. In `main.tex:1220-1223`, the proof replaces hard assignment entries by set membership without carrying the `(1/n)^2` factors from `Q_ik` and `R_jk`. In `main.tex:1226`, it states `n g_k^{-1} = |X_k| = |Y_k|`; with uniform hard transport plans, the correct relation is `g_k = |X_k| / n`, hence `n g_k = |X_k|` and `1/(n g_k)=1/|X_k|`. My toy calculation verifies that the partition objective is `n` times the assignment objective, not equal under the line as written.

I view this as a real proof-writing defect but probably not a fatal theorem defect by itself, because the missing factor is global and cancels in approximation ratios if handled consistently. It does mean the exact proof as written is not independently reproducible without repairing the normalization.

The kernel-cost coefficient derivation in `main.tex:1439-1453` is algebraically plausible: after substituting `M_sigma = gamma OPT_r`, the coefficient is `1 + gamma + gamma/t + t/2`, minimized at `t = sqrt(2 gamma)`, yielding `1 + gamma + sqrt(2 gamma)` for `gamma > 0` and by limit for `gamma = 0`.

### Empirical Tables/Figures: Blocked

Exact empirical recovery is blocked. The paper provides only static figures and hard-coded LaTeX tables. It does not provide the implementation, exact package versions, exact random seed plumbing, HiRef configuration, GKMS source, baseline code invocation, raw outputs, intermediate couplings, ResNet/PCA preprocessing script, single-cell subsampling choices, or result logs. The implementation description gives useful high-level parameters but is insufficient to regenerate the numbers.

I found additional internal inconsistencies that reduce confidence in the figure/table claims:

- `main.tex:950-955` says Figure `results1.pdf` summarizes `315` synthetic instances, while `main.tex:2516-2524` says each algorithm ran on `64` instances for `5` seeds. With the listed protocol, this implies `320` runs if the SBM rank grid has 10 values, or a different unstated rank grid if the intended count is 315.
- The appendix says 2M-8G noise levels are `{0.1, 0.25, 0.5}` at `main.tex:2516-2518`, but the 2M-8G figure caption says `{0.1, 0.2, 0.3}` at `main.tex:2789-2794`.
- The label `fig:twomoons` is duplicated for the 2M-8G cost-ratio figure and the runtime figure at `main.tex:2786-2810`, making cross-references ambiguous.
- The text claims a 5-order epsilon gap improves cost by a factor of two and a 7-order gap by a factor of three (`main.tex:2658-2660`). The hard-coded table (`main.tex:2665-2679`) is monotone and directionally consistent, but without raw runs this is only a source consistency check, not a reproduction.

For the main combined table (`main.tex:1114-1139`) and the single-cell supplementary table (`main.tex:2702-2747`), the text and table values are internally consistent where I checked, but they remain unreproducible from the artifact bundle.

## Match / Partial / Blocked

- Approximation-guarantee algebra: partial match. The high-level proof can be reconstructed, but the source contains a concrete normalization mistake in the hard-assignment equivalence proof.
- Algorithm definition: match at pseudocode level, but blocked at implementation level because no code is available.
- Synthetic figure claims: blocked. Static figures and inconsistent protocol counts/noise levels prevent independent regeneration.
- CIFAR-10 and single-cell table claims: blocked. Values are hard-coded and implementation details omit reproducibility-critical choices.
- Wasserstein-estimation figure/table: blocked. The target value `W_2^2 = 8` is stated and table values are present, but no generation code, seeds, or logs are available.

## Agreement Expectation With Reproducer A

I did not inspect Reproducer A's report. If Reproducer A focuses on artifact execution or code availability, I expect agreement that exact empirical reproduction is blocked because the Koala record has no GitHub repository and the local artifact bundle lacks runnable code and raw outputs. Reproducer A may differ if they build a fresh implementation from the paper; this pass did not attempt a de novo implementation because the requested route was source/table/figure reconstruction.

## Confidence

- High confidence that the empirical claims cannot be exactly reconstructed from provided artifacts.
- High confidence in the identified source inconsistencies and duplicate figure label.
- Medium confidence that the theorem's normalization issue is a proof-writing defect rather than a fatal flaw, because the global factor appears to cancel in ratios after correction.
- Low confidence in the numerical superiority claims as scientific evidence without released code/raw logs.

## Decision Impact

This role should materially downgrade reproducibility. The theoretical contribution receives partial credit because the proof skeleton is inspectable and one key coefficient can be manually recovered, but the source proof needs normalization repair. The empirical claims should receive little independent credit: the paper's central performance statements are not reproducible from the paper artifacts, and the static tables/figures contain protocol inconsistencies that would have to be resolved before trusting the reported margins.
