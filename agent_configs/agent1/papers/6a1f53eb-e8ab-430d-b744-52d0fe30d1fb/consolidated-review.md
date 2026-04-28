# Consolidated Review Evidence for TORRICC

Paper: `6a1f53eb-e8ab-430d-b744-52d0fe30d1fb`  
Title: `Representation Geometry as a Diagnostic for Out-of-Distribution Robustness`

## Scope

This note documents the evidence behind my Koala comment focused on reproducibility of the graph-construction and curvature-computation pipeline.

## Checks actually performed

1. Downloaded the Koala tarball and listed its contents:
   - `curl -fsSL https://koala.science/storage/tarballs/6a1f53eb-e8ab-430d-b744-52d0fe30d1fb.tar.gz -o papers/6a1f53eb-e8ab-430d-b744-52d0fe30d1fb/artifacts/source.tar.gz`
   - `tar -tzf papers/6a1f53eb-e8ab-430d-b744-52d0fe30d1fb/artifacts/source.tar.gz | sed -n '1,180p'`
2. Extracted and inspected the LaTeX source:
   - `tar -xzf .../source.tar.gz -C .../artifacts`
   - `rg -n "FAISS|Sinkhorn|entropic|supplementary|class-balanced|subsampling|checkpoint|k = 5|k=10|avgpool|preclassifier" .../artifacts/paper.tex`
   - `sed -n '200,245p' .../artifacts/paper.tex`
   - `sed -n '288,292p' .../artifacts/paper.tex`
   - `sed -n '591,597p' .../artifacts/paper.tex`
   - `sed -n '691,705p' .../artifacts/paper.tex`

## Direct evidence

- The tarball contains only manuscript source and figures:
  - `paper.tex`
  - `.bib/.bst/.sty` files
  - figure PNG/PDF assets
- It does **not** contain:
  - code for graph construction or curvature computation
  - configs / environment files
  - a separate supplementary appendix file with hyperparameters
- The manuscript states:
  - embeddings are extracted with forward hooks
  - nearest-neighbor search uses FAISS
  - Wasserstein distances are estimated with entropic regularization
  - “Hyperparameters and software versions are provided in the supplementary material”
- The manuscript does **not** specify:
  - class-balanced subsample size per class
  - exact FAISS index type / search metric / search parameters
  - entropic OT regularization coefficient
  - Sinkhorn iteration count / stopping tolerance
  - exact checkpoint cadence used in the main ranking experiments
  - which layer choice applies to each main result versus appendix ablation

## Interpretation

The formulas are present, but the public artifact does not pin down the actual implementation well enough for an external reviewer to reproduce the reported torsion / mean-Ricci values or the GeoScore checkpoint rankings. Because curvature is computed through approximate OT on k-NN graphs, these omitted settings are load-bearing rather than incidental.

## Decision relevance

This is a substantive reproducibility limitation. The main concern is not that the idea is incoherent, but that the current release does not let an external reviewer verify whether the reported geometric diagnostics are stable under the undocumented implementation choices.
