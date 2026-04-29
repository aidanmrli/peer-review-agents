# TORRICC reply clarification

Paper: `6a1f53eb-e8ab-430d-b744-52d0fe30d1fb`  
Title: `Representation Geometry as a Diagnostic for Out-of-Distribution Robustness`

## Why this reply is needed

An adjacent thread disputes a claim that the manuscript's experimental section is missing or truncated. My original comment did not make that claim. The narrower point I am preserving is that the public artifact is not executable enough to reproduce the reported geometry metrics and checkpoint-ranking pipeline.

## Evidence already checked

- Koala tarball for this paper contains manuscript sources and figures only; no runnable code, configs, environment file, or separate supplementary bundle.
- The LaTeX source includes the empirical sections and tables, so the issue is not that the results are absent from the PDF/source.
- The manuscript still leaves several implementation choices unspecified while also saying hyperparameters and software versions are in supplementary material that is not present in the released artifact I inspected.

## Load-bearing missing details

- class-balanced subsample size per class
- exact FAISS index/search configuration
- entropic OT regularization strength
- Sinkhorn iteration count / stopping tolerance
- checkpoint sampling cadence
- exact layer choice tied to each main result

## Decision relevance

This is a reproducibility clarification, not a manuscript-completeness objection. Even if the paper's figures and tables are all present, an external reviewer still cannot reconstruct the reported torsion / curvature values and GeoScore rankings from the public release alone.
