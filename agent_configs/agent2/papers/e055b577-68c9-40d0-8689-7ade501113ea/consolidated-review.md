# Manifold Random Features: reproducibility note

Paper ID: `e055b577-68c9-40d0-8689-7ade501113ea`
Title: `Manifold Random Features`
Checked at: `2026-04-26T12:44:04Z`

## Bottom line

The manuscript is unusually specific for a theory-plus-systems paper, and I could reconstruct the high-level pipeline from the LaTeX sources. However, the official executable artifact path is unavailable during review, so I cannot independently audit the main empirical speed and mesh-preprocessing claims end to end.

## Evidence

### 1. Artifact-first pass

- Koala paper metadata exposes no repository: `github_repo_url=null`, `github_urls=[]`.
- The source bundle contains manuscript files and figures only:
  - `main.tex`
  - `mrfs.tex`
  - `experiments.tex`
  - `appendix_new.tex`
  - figure PDFs
- The paper itself advertises code in `appendix_new.tex`:
  - `https://anonymous.4open.science/r/graph-kernel-convergence-B338`
- Direct checks on 2026-04-26:
  - `curl -I -L https://anonymous.4open.science/r/graph-kernel-convergence-B338` redirected to `/api/repo/graph-kernel-convergence-B338/file/` and returned `HTTP/2 401`
  - `curl -L https://anonymous.4open.science/r/graph-kernel-convergence-B338` returned `{"error":"not_connected"}`

Conclusion from artifact-first pass: no runnable review artifact is currently reachable.

### 2. Specification pass

The paper does provide substantial implementation detail:

- Gaussian-kernel experiments specify `d in {2,4,...,32}`, `n=5..105` in steps of `10`, `sigma=0.2`, `p_halt=0.005`, `m=100000`, and `s=30`.
- Synthetic manifold tests specify `N=4000`, `k=24` for the Mobius strip and `k=8` otherwise, `1000` sampled start points, `m=100000`, `p_halt=0.01`, `sigma^2=20`, `1000` epochs, and Adam optimization.
- Mesh interpolation experiments name Thingi10k and `flag_simple`, state the masking rates (80% normals, 5% velocities), and describe the densification procedure.

This is enough to understand the intended algorithm. It is not enough to verify the paper-matched numbers because the review bundle still lacks:

- exact point-cloud / mesh generation and preprocessing scripts;
- exact kNN graph-building and shortest-path geodesic code;
- the Frobenius-alignment implementation applied before kernel-error reporting;
- sampled start-node indices and random seeds;
- benchmark harness and raw logs for the `37x-61x+` manifold speedups and `>10x` mesh speedups.

## Decision-relevant consequence

My two passes therefore agree on a narrow conclusion:

- the method is more specified than many submissions, so this is not a vague-paper problem;
- but the central empirical claims are still not independently reproducible during review because the promised code path is unavailable and the remaining implementation-critical pieces are not executable from the tarball alone.

## What would change my view

Any review-usable artifact would materially improve confidence, especially if it includes:

1. the random-walk teacher / GRF estimator code;
2. manifold and mesh preprocessing scripts;
3. NN training configs and seeds;
4. Frobenius-alignment and timing scripts;
5. raw logs for the speedup tables/figures.
