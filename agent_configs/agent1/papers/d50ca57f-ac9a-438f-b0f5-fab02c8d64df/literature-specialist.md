# Literature Specialist Report

Paper: **Transport Clustering: Solving Low-Rank Optimal Transport via Clustering**  
Koala paper ID: `d50ca57f-ac9a-438f-b0f5-fab02c8d64df`  
Role: Literature Specialist  
Date: 2026-04-24

## Scope and Evidence Protocol

I evaluated whether the paper's novelty and framing are accurate relative to primary prior work available before or at release. I used the paper's local LaTeX/PDF artifacts and primary publication pages only; I did not use reviews, decisions, citation counts, social media, or future impact signals.

Local evidence inspected:

- `artifacts/main.tex`
- `artifacts/references.bib`
- `artifacts/00README.json`
- Koala `get_paper` metadata for `d50ca57f-ac9a-438f-b0f5-fab02c8d64df`

Commands and checks used:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,260p'
sed -n '1,220p' skills/literature-specialist.md
rg -n "low-rank|Forrow|Scetbon|Latent|FRLC|K-means|Monge|Kantorovich|registration|clustering|approx" artifacts/main.tex
rg -n "Laclau|Genevay|Wasserstein|liu2021sparse|Solomon2016|balanced|size constrained" artifacts/main.tex artifacts/references.bib
nl -ba artifacts/main.tex | sed -n '180,285p'
nl -ba artifacts/main.tex | sed -n '364,455p'
nl -ba artifacts/main.tex | sed -n '470,648p'
nl -ba artifacts/main.tex | sed -n '671,735p'
nl -ba artifacts/main.tex | sed -n '1074,1170p'
```

`pdftotext` was not available in the environment, so source-line locations below refer to the local LaTeX source. The Koala record reports no linked GitHub repository (`github_urls: []`), so this role did not inspect author code.

## Novelty Claim Checked

The paper claims that low-rank OT can be reduced to generalized K-means after a full-rank transport registration step, yielding a simple transport clustering algorithm with constant-factor approximation guarantees. The exact claims appear in the abstract and introduction at `main.tex:182-199` and `main.tex:275-284`; the formal construction is at `main.tex:470-606`; the Kantorovich extension is at `main.tex:671-678`; and the approximation framing is at `main.tex:691-731`.

My conclusion: **the central reduction appears genuinely distinct from the main low-rank OT solver literature, but the surrounding framing is incomplete.** The paper correctly cites the low-rank OT lineage, but it under-discusses prior OT co-clustering and OT-based clustering work, and it should be more explicit that generalized K-means and the K-means connection were already established by Scetbon and Cuturi (2022).

## Prior Work Considered

Core low-rank OT:

- Forrow et al. (2019), *Statistical Optimal Transport via Factored Couplings*, AISTATS/PMLR. This introduced transport rank/factored couplings for high-dimensional OT estimation and robustness. Primary source: https://proceedings.mlr.press/v89/forrow19a.html
- Scetbon, Cuturi, and Peyre (2021), *Low-Rank Sinkhorn Factorization*, ICML/PMLR. This gives the explicit `Q diag(1/g) R^T` low-nonnegative-rank formulation and alternating/mirror-descent solver for arbitrary costs. Primary source: https://proceedings.mlr.press/v139/scetbon21a.html
- Scetbon and Cuturi (2022), *Low-rank Optimal Transport: Approximation, Statistics and Debiasing*, NeurIPS/OpenReview PDF. This establishes approximation/statistical properties and explicitly links low-rank OT bias to generalized K-means and K-means. Primary source: https://openreview.net/pdf?id=4btNeXKFAQ
- Lin, Azabou, and Dyer (2021), *Making transport more robust and interpretable by moving data through a small number of anchor points*, ICML/PMLR. This is LatentOT, using source/target anchors to regularize rank and improve robustness/interpretablity. Primary source: https://proceedings.mlr.press/v139/lin21a.html
- Halmos et al. (2024), *Low-Rank Optimal Transport through Factor Relaxation with Latent Coupling*, NeurIPS/OpenReview. This introduces FRLC from latent coupling factorization, supports multiple OT objectives and marginal regimes, and overlaps directly with the paper's empirical baseline set. Primary source: https://openreview.net/forum?id=hGgkdFF2hR
- Liu et al. (2021), *Approximating Optimal Transport via Low-rank and Sparse Factorization*. This is relevant because it explicitly notes that Monge OT plans are often full rank and proposes low-rank plus sparse approximations. Primary source: https://arxiv.org/abs/2111.06546

Clustering, co-clustering, and approximation algorithms:

- Peng and Wei (2007), *Approximating k-means-type clustering via semidefinite programming*, for SDP formulations of K-means-type objectives. Primary source: https://optimization-online.org/2005/04/1114/
- Arthur and Vassilvitskii (2007), *k-means++*, for logarithmic competitive seeding guarantees. Primary source: https://theory.stanford.edu/~sergei/papers/kMeansPP-soda.pdf
- Kumar, Sabharwal, and Sen (2004), `(1+epsilon)` approximation for K-means, as cited by the paper.
- Kolliopoulos and Rao (2007), Euclidean K-medians approximation scheme, as cited by the paper.
- Laclau et al. (2017), *Co-clustering through Optimal Transport*, ICML/PMLR. This is a direct missing related-work citation: it uses entropy-regularized OT between instance and feature measures, then factorizes the optimal coupling to obtain co-clusters. Primary source: https://proceedings.mlr.press/v70/laclau17a.html
- Genevay, Dulac-Arnold, and Vert (2019), *Differentiable Deep Clustering with Cluster Size Constraints*. This rewrites K-means as an OT task with entropic regularization and size constraints. Primary source: https://research.google/pubs/differentiable-deep-clustering-with-cluster-size-constraints/
- Zhuang, Chen, and Yang (2022), *Wasserstein K-means for clustering probability distributions*, NeurIPS. This is relevant to "OT clustering" framing, though it clusters distributions rather than solving low-rank point-to-point OT. Primary source: https://proceedings.neurips.cc/paper_files/paper/2022/hash/4a1d69d1f64c6b6df105b15984ca527a-Abstract-Conference.html

Monge/Kantorovich registration and correspondence:

- Monge/Kantorovich OT itself is correctly cited and used as background (`main.tex:214-216`, `main.tex:294-352`).
- Solomon et al. (2016), *Entropic Metric Alignment for Correspondence Problems*, is in the paper's bibliography but not discussed in the body. It is relevant to correspondence/alignment framing, though it concerns entropic Gromov-Wasserstein correspondence rather than this paper's registered-cost reduction.
- Haker et al. (2004), *Optimal Mass Transport for Registration and Warping*, is a classical OT registration reference. It is not technically the same as the paper's algebraic cost registration, but the paper should distinguish its use of "registration" from this older usage.

## Overlap Versus Distinction

### Low-rank OT solvers

The paper's background is substantially accurate. Forrow et al. introduced the transport-rank/factored-coupling viewpoint; Scetbon et al. made the general low-rank Sinkhorn factorization explicit; Lin et al. introduced an anchor/latent OT factorization; and FRLC introduced a latent-coupling parameterization and stronger solver flexibility. The current paper cites these works in the relevant background (`main.tex:232-243`, `main.tex:364-398`) and compares against LOT, FRLC, and LatentOT in synthetic experiments (`main.tex:1078-1082`, `main.tex:2513-2515`).

The distinction is that prior solvers optimize the low-rank factors directly, generally through local nonconvex routines, while this paper fixes a full-rank Monge/Kantorovich correspondence and then solves a single registered generalized K-means problem (`main.tex:532-606`). I did not find a prior primary source, among the checked low-rank OT papers, that states this specific reduction or the claimed Monge-registered constant-factor approximation. This is a real novelty point.

### K-means and generalized K-means

The paper's strongest overlap is with Scetbon and Cuturi (2022). That prior work already introduced the generalized K-means formulation in the low-rank OT context and proved that, for self-transport with squared Euclidean cost, K-means is recovered. The current paper correctly attributes generalized K-means to Scetbon and Cuturi (`main.tex:429-452`) and uses that connection as the base object. Therefore, the novel contribution is not "LR-OT generalizes K-means" but rather **using a full-rank transport map to turn the two-sided low-rank OT co-clustering problem into a one-sided generalized K-means instance with approximation guarantees**.

The abstract and introduction sometimes blur this distinction by presenting "generalizes K-means to co-clustering" as part of the paper's motivating contribution (`main.tex:185-187`, `main.tex:241-243`). This should be tightened because the K-means/generalized K-means bridge is prior work.

### Monge and Kantorovich registration

The Monge registration step is the paper's clearest conceptual novelty: choose the optimal full-rank Monge permutation, register the cost as `C P_sigma^T`, solve generalized K-means in one factor, and recover the second factor by conjugation (`main.tex:580-606`). This is distinct from ordinary OT registration or correspondence estimation because the OT map is not the final output; it is used to transform the low-rank OT optimization.

The Kantorovich registration extension is weaker. It is presented as an analogue for arbitrary marginals and unequal sizes (`main.tex:671-678`) and empirically tested (`main.tex:2636-2654`), but the main theoretical claims are explicitly for Monge registration. The paper should frame Kantorovich registration as a heuristic or partially justified extension unless a matching approximation theorem is provided.

### Approximation algorithms

The approximation contribution is not a new clustering approximation algorithm. The paper imports existing K-means/K-medians approximation machinery (`main.tex:632-646`, `main.tex:920-924`) after proving that the registered proxy has bounded cost relative to low-rank OT. That framing is acceptable, but the implementation uses approximate registration via Sinkhorn or HiRef (`main.tex:2478-2484`) rather than exact full-rank OT. The literature context should explicitly separate:

1. the theoretical pipeline with exact Monge registration; and
2. the practical pipeline using entropic or hierarchical approximate OT, where the guarantee can degrade.

The entropy sensitivity ablation (`main.tex:2656-2679`) supports this concern: worse full-rank registration materially worsens the low-rank cost. This is not a novelty failure, but it is decision-relevant for how strongly the approximation guarantee supports the empirical method.

## Missing Citations or Baselines

1. **Missing related work: Co-clustering through OT.** Laclau et al. (2017) is the most important omission. It is not a low-rank OT solver, but it directly studies co-clustering through an OT coupling and then factorizes the coupling into row/column partitions. Because this paper repeatedly frames its method as transport clustering and co-clustering (`main.tex:192-193`, `main.tex:470-476`, `main.tex:507-527`), Laclau et al. should be cited and distinguished.

2. **Missing OT clustering framing.** Genevay et al. (2019) and Zhuang et al. (2022) are not direct baselines for low-rank OT cost, but they are relevant to claims about clustering via OT and K-means/OT equivalences. They should appear in related work to prevent the impression that the OT-clustering connection begins with low-rank OT.

3. **Low-rank plus sparse OT caveat.** Liu et al. (2021) is in `references.bib` but not cited in the body. It directly supports and complicates the paper's claim that Monge plans are full rank (`main.tex:218-227`): if the target is approximating arbitrary full-rank OT, low-rank-only constraints can be inaccurate. The paper should explicitly state that its goal is solving the low-rank OT objective and extracting structure, not approximating every full-rank OT plan.

4. **Baseline omissions are context-dependent.** For low-rank OT objective value, the reported baselines LOT, FRLC, LatentOT, and FactoredOT cover the core lineage reasonably well (`main.tex:1078-1082`, `main.tex:1164-1166`, `main.tex:2513-2515`). For broad co-clustering claims, however, an OT co-clustering baseline such as Laclau et al. would be appropriate, at least on compatible tabular/co-clustering data. For CIFAR and single-cell real experiments, LatentOT and FactoredOT are not included; the paper says it uses methods "which scale to it" (`main.tex:1141`) but should document the scaling exclusion with runtime/memory or implementation constraints.

5. **Balanced/size-constrained clustering literature is under-discussed.** The hard low-rank formulation imposes matched co-cluster sizes (`main.tex:499-513`), which overlaps with balanced or size-constrained clustering. The paper has a `bertoni2012size` entry in the bibliography but does not discuss this body of work. This is a minor omission for low-rank OT novelty, but relevant to the hard-assignment framing.

6. **Registration terminology should be disambiguated.** OT-based image/shape registration and correspondence methods predate this work. The paper's "registration" is an algebraic cost registration for a low-rank OT reduction, not a standard registration method. A short distinction from OT registration/correspondence literature would make the novelty claim cleaner.

## Accuracy of Framing

Accurate:

- Low-rank OT as nonnegative-rank constrained OT is correctly positioned.
- Prior low-rank solvers are correctly characterized as nonconvex local solvers without global approximation guarantees beyond stationary convergence.
- The paper fairly identifies Scetbon and Cuturi as the source of generalized K-means in the low-rank OT setting.
- The experiments compare against the main low-rank OT solver families for the objective the paper optimizes.

Overstated or incomplete:

- The K-means/co-clustering narrative is somewhat overclaimed unless the paper more explicitly says that this connection is inherited from Scetbon and Cuturi (2022).
- The phrase "transport clustering" risks collision with existing OT clustering/co-clustering work unless Laclau et al., Genevay et al., and Wasserstein K-means are cited and separated.
- The Kantorovich registration discussion is under-theorized relative to the confidence of the broader wording.
- The approximation guarantee is easier to overread than it should be, because practical experiments use approximate full-rank registration.

## Decision Impact

From a literature perspective, this paper has a legitimate and nontrivial contribution: the Monge-registered reduction of low-rank OT to generalized K-means is not merely a repackaging of Forrow, Scetbon, LatentOT, or FRLC. That supports acceptance if the theory and experiments are correct.

The literature weaknesses are moderate, not fatal. The missing OT co-clustering and OT clustering citations weaken the breadth of the related-work framing, and the baseline set is incomplete for the broad "transport clustering/co-clustering" narrative. They do not invalidate the core low-rank OT novelty, but they should materially temper a strong-accept case. My literature-only score impact is: **positive for novelty (+), negative for incomplete framing and missing baselines/citations (-), net around weak accept if correctness and reproducibility hold; lean weak reject if the other roles find the approximation or empirical claims unreproducible.**
