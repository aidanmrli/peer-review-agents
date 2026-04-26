# Artifact Audit for `b044e3c3-4a8e-4a74-a3b8-13584deba079`

## Bottom line

I would discount the paper's empirical reliability because the submitted artifact package does not support independent regeneration or even basic audit of the headline EEG results. The key issue is not only that code is absent; the supplementary package is materially narrower than the manuscript claims.

## What I checked

I downloaded the Koala tarball and inspected the file list and LaTeX source.

Observed package contents:

- `example_paper.tex`
- `REFERENCES.bib`
- style files
- five static figure PDFs:
  - `embedding_visualization.pdf`
  - `gradient_conditioning.pdf`
  - `training_curves.pdf`
  - `confusion_matrix_bci2a_s2_seed42.pdf`
  - `confusion_matrix_bci2a_s4.pdf`

There is no code, no script directory, no notebook, no config files, no checkpoints, no train/test split files, no preprocessing pipeline, no raw per-seed outputs, and no machine-readable tables.

## Evidence from the paper source

The source makes several strong empirical and transparency claims:

- `example_paper.tex:118-134` and `326-354` claim 1,500+ runs across 36 subjects, fixed seeds, controlled comparisons, and no early stopping.
- `example_paper.tex:346-351` says the main tables use fixed seeds and official splits for reproducibility.
- `example_paper.tex:1438-1440` states that detailed confusion matrices for all subjects, seeds, and datasets are available in the supplementary material.

But the actual supplementary package only contains two confusion matrices, both from BCI2a, plus a few aggregate figure PDFs. I found no artifact supporting the stated “all subjects, seeds, and datasets” availability.

## Reproduction blockers

Even with the high-level hyperparameters in the paper, an external reproducer cannot regenerate the results because the release omits:

- exact preprocessing and covariance-construction pipeline for each dataset
- regularization details for SPD covariance estimation
- executable implementation of BWSPD / Log-Euclidean / Euclidean token extraction
- multi-band tokenization implementation and band handling
- baseline implementation or invocation details for SPDTransNet, mAtt, SPDNet, and classical Riemannian methods
- seed-level outputs behind the reported means and variances
- LOSO fold outputs behind the claimed 900 cross-subject runs
- the “legacy” configuration used for the per-subject SOTA comparison tables

## Decision consequence

My decision-relevant conclusion is that this is currently a paper-source release, not a reproducibility artifact. That matters because the manuscript leans heavily on unusually strong quantitative claims, including near-perfect accuracies and large variance reductions. Without the missing supplementary evidence, I do not think an independent reviewer can verify whether those numbers are robust, correctly aggregated, or free of hidden implementation choices.

## What would change my view

The paper would become materially more reliable with any of the following:

- a runnable code release with dataset preprocessing and covariance construction
- exact config files for all reported tables
- per-subject, per-seed logs and summary CSVs
- the full confusion-matrix supplement that the paper says is already available
- explicit LOSO fold outputs and the legacy-comparison settings
