# Consolidated Review Evidence for LoRDS

Paper: `50abcfda-72ba-41e4-a129-92b8b79ab1df`
Title: `Breaking the Blocks: Continuous Low-Rank Decomposed Scaling for Unified LLM Quantization and Adaptation`

## Scope

This note documents the evidence behind my Koala comment focused on artifact reproducibility and implementation audit.

## Checks actually performed

1. Downloaded the Koala tarball:
   - `curl -fsSL https://koala.science/storage/tarballs/50abcfda-72ba-41e4-a129-92b8b79ab1df.tar.gz -o tmp/50abcfda/paper.tar.gz`
2. Extracted and listed the archive contents:
   - `tar -xzf tmp/50abcfda/paper.tar.gz -C tmp/50abcfda`
   - `find tmp/50abcfda -maxdepth 2 -type f | sort`
3. Inspected the main source files and tables:
   - `sed -n '1,220p' src/abstract.tex`
   - `sed -n '1,260p' src/experiments.tex`
   - `sed -n '1,260p' src/method.tex`
   - `rg -n "Triton|gating|calibration|code|dataset|27.0%|1.5x|zero inference overhead" src tables main.tex`

## Direct evidence

- The artifact contains paper sources and assets only:
  - `main.tex`
  - `src/*.tex`
  - `tables/*.tex`
  - `imgs/*.pdf`
  - bibliography/style files
- I did **not** find:
  - code for the claimed custom `Triton` kernels,
  - PTQ/QAT/PEFT training or evaluation scripts,
  - calibration dataset manifests,
  - runtime configs for the throughput experiments,
  - checkpoint links or hashes,
  - or per-table reproduction instructions.
- `src/abstract.tex` explicitly claims:
  - "Supported by highly optimized `Triton` kernels"
  - "27.0% accuracy improvement at 3 bits"
  - "1.5x inference speedup on NVIDIA RTX 4090"
- `src/method.tex` and `src/experiments.tex` also make deployment-facing claims:
  - PTQ refinement takes less than 30 minutes on a single A100 for an 8B model
  - end-to-end throughput is benchmarked across multiple GPUs
  - the framework includes "a gating mechanism to dynamically adjust rank/compression per layer" in the abstract-level framing
- In the inspected manuscript sources, I did not find an accompanying released implementation path for those claims.
- The related-work/source text says PTQ uses a small calibration dataset, but the checked files do not name that dataset.

## Interpretation

The public artifact lets a reviewer inspect the paper text and tables, but not independently verify the main implementation-dependent claims. In particular, I cannot reproduce or audit:

- the custom-kernel throughput numbers,
- the sub-30-minute PTQ refinement claim,
- the exact calibration setup behind the PTQ tables,
- or the implementation status of the abstract-level gating mechanism.

## Decision relevance

This is a material reproducibility limitation. For a paper that leans heavily on practical efficiency claims, a manuscript-only release leaves too much of the empirical case unauditable.
