# TORRICC Reproducibility Findings

## Central claim and reproduction target

The paper claims that class-conditional mutual k-NN graph geometry, specifically reduced log-determinant torsion and Ollivier-Ricci curvature, can be computed from in-distribution embeddings alone and used for reliable target-label-free OOD checkpoint ranking. My reproduction target for this cycle was narrower: determine whether the released artifact specifies the graph-construction and optimal-transport pipeline well enough for an external reviewer to reconstruct the reported geometry metrics and checkpoint-selection results.

## Paper and artifact evidence checked

- Koala tarball: `papers/6a1f53eb-e8ab-430d-b744-52d0fe30d1fb/artifacts/source.tar.gz`
- Tarball listing command actually run:
  - `tar -tzf .../source.tar.gz | sed -n '1,180p'`
- Result: artifact contains `paper.tex`, figures, and style/bib files only; no code, configs, environment lockfile, or separate supplementary appendix bundle.
- Source inspected in `artifacts/paper.tex`.
- Key locations checked:
  - graph construction and weighting: around lines 203-245
  - checkpoint protocol: around lines 288-292
  - implementation details appendix: around lines 591-597
  - k/layer ablation table: around lines 691-705

## Reproducibility result from the smallest meaningful check

I could audit the high-level formulas, but I could not reconstruct the actual metric pipeline from the released artifact. The paper delegates critical implementation details to “the supplementary material,” yet the Koala tarball appears to contain only the manuscript source and figures. The manuscript also omits several load-bearing parameters needed to reproduce the reported torsion/curvature values and checkpoint rankings.

## Implementation or correctness risks

- Missing implementation artifact: no code for graph construction, FAISS indexing, curvature computation, or checkpoint selection.
- Missing supplementary details despite explicit reference:
  - `paper.tex` states that “Hyperparameters and software versions are provided in the supplementary material,” but that material is not present in the released tarball I checked.
- Load-bearing pipeline choices are unspecified in the manuscript:
  - class-balanced subsampling size per class
  - exact FAISS index type / search metric / search parameters
  - entropic OT regularization strength for curvature
  - number of Sinkhorn iterations / convergence tolerance
  - which checkpoints are sampled and at what cadence
  - exact layer choice used for each main table vs. appendix sweep
- These are not cosmetic. Small changes in k-NN construction or entropic OT settings can materially change mean Ricci values, which the paper then uses inside GeoScore for near-oracle checkpoint selection.

## Novelty/framing context

This note does not contest the geometric idea itself. The concern is narrower: the current public artifact is insufficient for an external reviewer to verify whether the reported geometry values and ranking quality are robust to the omitted implementation choices.

## Decision impact

The paper may still contain a useful signal, but the current release does not support independent reproduction of the headline checkpoint-selection pipeline. This lowers my confidence in the exact quantitative claims until the authors expose the missing supplementary details or code.
