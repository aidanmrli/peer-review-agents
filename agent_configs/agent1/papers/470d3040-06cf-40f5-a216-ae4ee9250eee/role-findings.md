# Role Findings: 470d3040-06cf-40f5-a216-ae4ee9250eee

## Reproducibility lead
- Central claim reviewed: MUNKEY delivers deployment-oriented zero-shot unlearning with negligible overhead by deleting per-instance memory entries instead of updating weights.
- Reproduction target: recover the accounting needed to reproduce the claimed deployment efficiency, including memory-bank size, inference cost, and deletion/update cost.

## Reproducer A
- Artifact-first check: the Koala tarball contains LaTeX sources and figures only (`main_arxiv.tex`, `Sections/*.tex`, `Figures/*`, `00README.json`), with no training code, inference code, index-building code, or runtime scripts.
- Result: could not verify any runtime or deployment claim from executable artifacts.

## Reproducer B
- Clean-room spec pass from paper text:
- Method defines one fixed key and one learnable exemplar token per training instance: `M = {(k_i, v_i)}_{i=1}^N` in `Sections/method.tex`.
- Main experiments use token dimension 128, `K=4` neighbors, a ViT-B/16 frozen key encoder, and ensemble aggregation during inference in the main architecture (`Sections/appendix.tex` hyperparameters; `Sections/results.tex` aggregation discussion).
- Result: spec is sufficient to infer linear memory growth and multi-pass retrieval-conditioned inference, but not sufficient to reproduce serving cost.

## Implementation auditor
- Runtime table in `Sections/results.tex` reports only `Unlearn` and `Train/Ep.`; there is no inference-latency table, no ANN index build/update cost, and no memory-footprint report.
- The paper claims deletion is "near-instantaneous even at scale," but the accounting excludes the cost of maintaining and querying the memory/index used at serving time.
- Appendix acknowledges that ensemble aggregation requires `K` separate inference passes; main text says ensemble is the primary architecture and `K=4` is used in the main experiments.

## Correctness specialist
- The deployment claim is directionally plausible for deletion itself, but incomplete as an end-to-end systems claim because cost is shifted rather than eliminated.
- Since each sample owns a stored key-token pair and inference retrieves neighbors from the current bank, scale-sensitive quantities are load-bearing: storage growth with `N`, retrieval/index update cost after deletions, and serving latency under `K=4` ensemble inference.
- Without those measurements, "deployment-oriented" should be read as deletion-time only, not full lifecycle efficiency.

## Literature specialist
- Existing thread already covers RAC lineage, Ready2Unlearn, backbone leakage, and novelty calibration.
- Distinct addition here: the paper needs deployment accounting, not just deletion-time accounting, to support its practical framing.
