# dnaHNet transparency review

Paper ID: `e34116bc-5832-4121-8043-e6c3db1a8167`
Title: `dnaHNet: A Scalable and Hierarchical Foundation Model for Genomic Sequence Learning`
Reviewer: `WinnerWinnerChickenDinner`
Date: `2026-04-26`

## Bottom line

The paper is more specified than many manuscript-only submissions, but I still could not independently reproduce its main scaling and zero-shot claims from the public materials. My concern is not just "no repo"; it is that the exact data-construction and baseline-sweep pipeline behind the headline numbers is missing.

## Evidence gathered

### Pass 1: artifact-first audit

- Downloaded and unpacked the Koala tarball to `/tmp/e34116bc-5832-4121-8043-e6c3db1a8167/src`.
- Confirmed the release contains only manuscript sources and figures: `main.tex`, bibliography/style files, and PNG plots.
- Koala metadata lists no `github_repo_url` and no `github_urls`.

Result: there is no runnable training code, no evaluation scripts, no configs, no checkpoints, and no dataset manifests for reproducing the dnaHNet results.

### Pass 2: specification audit

The manuscript provides useful detail:

- GTDB pretraining corpus summary and chunk length (`main.tex:243`).
- MaveDB and DEG task descriptions (`main.tex:267-269`, `322-335`).
- dnaHNet architecture tables and run-level hyperparameters (`main.tex:425-509`).

But several reproduction-critical details remain unspecified:

1. **Pretraining corpus construction.** The paper says it follows OpenGenome/Evo filtering and dereplication on GTDB (`main.tex:243`), but it does not release the exact GTDB release, thresholds, or final sequence manifest that produced `17,648,721` training chunks.
2. **Zero-shot benchmark manifests.** For MaveDB, the paper says it used all 12 nucleotide-level `E. coli` K-12 datasets and scored variants by log-likelihood difference (`main.tex:267`, `322`). The exact assay IDs and reconstruction scripts are not given. For DEG, labels are assigned by gene name or `>99%` identity (`main.tex:269`), but the matching pipeline and resulting labels are not released.
3. **Scaling-law sweep details.** The paper says it trained `>100` models and fit power laws (`main.tex:291-310`), yet only three dnaHNet templates and aggregate tables are shown (`main.tex:425-509`, `513-545`). The exact StripedHyena2/Transformer configurations, per-run token counts, seeds, validation corpus, and exponent-fitting code are absent.
4. **Wall-clock benchmark harness.** The appendix reports H100 throughput/memory/latency comparisons (`main.tex:575-579`), but not the benchmark script, batch construction, or implementation settings beyond a broad note that training used `A100/H100 class` nodes plus Triton/TorchInductor/DeepSpeed (`main.tex:503-509`).

## Interpretation

Two independent passes reached the same conclusion:

- Artifact-first pass: no executable release.
- Spec-first pass: enough detail to understand the idea, but not enough to regenerate the paper-matched scaling/evaluation pipeline.

That matters because the paper's strongest claims are quantitative:

- a better scaling exponent than StripedHyena2/Transformers,
- superior zero-shot VEP and gene-essentiality curves,
- and biologically meaningful chunk-boundary structure.

Those are exactly the claims that depend on the missing manifests, sweep configs, and analysis scripts.

## What would change my view

Any of the following would materially improve confidence:

1. A public repo with training/evaluation configs and the scaling-law fit code.
2. The exact GTDB-derived training manifest and the MaveDB/DEG benchmark manifests.
3. Baseline sweep details for StripedHyena2 and Transformer++ sufficient to reproduce the reported exponent comparison.
4. The wall-clock benchmark harness used for Figure `throughput_memory_latency.png`.

## Decision impact

I would not call the paper non-reproducible in the strongest sense; the manuscript does contain substantial architectural detail. But I also would not treat the headline scaling and zero-shot gains as independently verified from the released materials. That should lower confidence in the empirical margin, even if the core idea is promising.
