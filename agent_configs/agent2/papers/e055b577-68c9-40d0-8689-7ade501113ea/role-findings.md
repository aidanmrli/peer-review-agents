## Reproducibility lead: central claim and reproduction target

Central target: reproduce the paper's claim that Manifold Random Features approximate manifold heat / diffusion kernels accurately while delivering large inference-time speedups over spectral baselines on synthetic manifolds and mesh interpolation tasks.

Bottom line: the manuscript is much more specific than average, but the official executable artifact is not accessible during review, so the main empirical claims are not independently auditable end to end.

## Reproducer A: artifact-first check

- Koala metadata lists no GitHub artifact: `github_repo_url=null`, `github_urls=[]`.
- The source tarball contains LaTeX sources and figures only (`main.tex`, `mrfs.tex`, `experiments.tex`, `appendix_new.tex`, PDFs), not runnable code, configs, or logs.
- The manuscript advertises code in Appendix (`appendix_new.tex`): `https://anonymous.4open.science/r/graph-kernel-convergence-B338`.
- Direct check on 2026-04-26:
  - `curl -I -L https://anonymous.4open.science/r/graph-kernel-convergence-B338` redirects to `/api/repo/graph-kernel-convergence-B338/file/` and returns HTTP 401.
  - `curl -L https://anonymous.4open.science/r/graph-kernel-convergence-B338` returns `{"error":"not_connected"}`.
- Result: no review-usable implementation path for the random-walk estimator, manifold preprocessing, NN training, or timing harness.

## Reproducer B: clean-room/specification check

- Positive: many core hyperparameters are explicit.
  - Gaussian-kernel grid test: `d in {2,4,...,32}`, `n=5..105 step 10`, `sigma=0.2`, `p_halt=0.005`, `m=100000`, `s=30`.
  - Synthetic manifolds: `N=4000`, `k=24` for Mobius and `k=8` otherwise, `1000` start points, `m=100000`, `p_halt=0.01`, `sigma^2=20`, `1000` epochs, Adam.
  - Mesh experiments: Thingi10k normals with 80% masking; `flag_simple` velocity with 5% masking and explicit densification description.
- Remaining blockers for faithful reproduction:
  - exact manifold point-cloud generation and mesh preprocessing scripts;
  - kNN graph construction details beyond high-level prose;
  - shortest-path geodesic approximation implementation;
  - Frobenius-alignment routine used before kernel error reporting;
  - exact timing scripts / hardware harness behind the `37x-61x+` and `>10x` claims;
  - training seeds, sampled start-node indices, and raw per-run logs.
- Result: I can reconstruct the broad method, but not verify the paper-matched numbers or timing tables.

## Implementation auditor: code/artifact/repo match

- Paper claims executable code exists, but the only linked repo is unavailable.
- No code in tarball to confirm that the implementation matches the stated random-walk teacher, NN surrogate, or baseline spectral pipeline.
- The paper therefore currently fails the minimum artifact audit for reproducibility-focused review.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

- I did not identify an obvious mathematical contradiction from the manuscript skim.
- Main correctness risk for review is empirical auditability rather than theorem validity: the strongest quantitative claims are speed/accuracy tradeoffs that depend on unavailable preprocessing and benchmarking code.
- The use of Frobenius alignment before kernel-error reporting is sensible but important enough that the exact implementation should be exposed.

## Literature specialist: novelty/framing against permitted prior work

- The paper appears novel in connecting GRFs on discretized manifolds to continuous manifold random features and in recovering positive/bounded Gaussian-kernel RFs through graph random walks.
- My comment should stay on reproducibility, not novelty ranking.
