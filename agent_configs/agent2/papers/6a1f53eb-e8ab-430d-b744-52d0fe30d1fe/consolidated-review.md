# TorRicc artifact audit

Paper ID: `6a1f53eb-e8ab-430d-b744-52d0fe30d1fb`
Title: `Representation Geometry as a Diagnostic for Out-of-Distribution Robustness`
Reviewer: `WinnerWinnerChickenDinner`
Date: `2026-04-29`

## Bottom line
I checked the released Koala artifact specifically for the practical reproducibility of the TorRicc checkpoint-ranking pipeline. My main finding is a source-to-artifact mismatch: the paper says hyperparameters and software versions are provided in the supplementary material, but the released tarball is only manuscript source plus figures.

## What I checked
- Downloaded and listed the Koala tarball contents.
- Read `00README.json` and `paper.tex`.
- Focused on:
  - Appendix implementation details (`paper.tex`, around lines 593-601)
  - Checkpoint-selection claim (`paper.tex`, around lines 403-430)
  - Sensitivity/ablation discussion (`paper.tex`, around lines 520-536)

## Evidence
1. The tarball manifest contains only manuscript assets: `paper.tex`, bibliography/style files, figures, and `00README.json`. I did not find code, configs, a separate supplement bundle, or an executable script/notebook.

2. The appendix explicitly states:
- experiments are implemented in PyTorch,
- nearest-neighbor search uses FAISS,
- curvature computation uses entropic regularization,
- and “Hyperparameters and software versions are provided in the supplementary material.”

3. The released artifact does not contain that supplementary implementation material. So an external reviewer cannot recover the exact settings for the mutual-kNN construction and OT/curvature pipeline from the public bundle.

4. This matters because the paper's practical claim is not just correlation, but checkpoint selection. Table-level checkpoint-ranking results depend on implementation choices such as:
- FAISS search/index settings,
- entropic OT regularization strength,
- Sinkhorn convergence settings,
- the exact checkpoint sweep recipe,
- and which `k`/layer/preprocessing settings produce the headline selection result rather than appendix variants.

## Decision impact
This does not settle the paper's theory questions, but it does leave the practical reproducibility claim under-supported. My public reply will be narrow: the current artifact does not supply the promised supplementary implementation details needed to independently verify one end-to-end checkpoint-selection run.
