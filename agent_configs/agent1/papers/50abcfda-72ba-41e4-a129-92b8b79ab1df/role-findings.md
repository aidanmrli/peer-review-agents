# Role Findings for LoRDS

Paper: `50abcfda-72ba-41e4-a129-92b8b79ab1df`
Title: `Breaking the Blocks: Continuous Low-Rank Decomposed Scaling for Unified LLM Quantization and Adaptation`

## Central claim and reproduction target

The paper claims a unified quantization/adaptation framework with practical deployment evidence: custom `Triton` kernels, less-than-30-minute PTQ refinement on an 8B model, 1.5x QLoRA throughput gains on RTX 4090, and strong PTQ/QAT/PEFT results.

## Paper and artifact evidence checked

- Downloaded and extracted the Koala tarball for `50abcfda-72ba-41e4-a129-92b8b79ab1df`.
- Inspected `src/abstract.tex`, `src/experiments.tex`, `src/method.tex`, and the PTQ/low-bit/PEFT/throughput tables.
- Grepped the source for `Triton`, `gating`, `calibration`, `LoftQ`, `NormalFloat`, `1.5x`, `27.0%`, and related implementation claims.

## Reproducibility result from the smallest meaningful check actually run

The Koala artifact is manuscript-only. I found LaTeX sources, style files, tables, and figures, but no runnable code, no Triton kernel sources, no training/evaluation scripts, no calibration dataset manifest, no benchmark config files, and no checkpoint links. This is enough to verify what the paper *claims*, but not enough to rerun or even inspect the decisive implementation path behind the PTQ refinement or throughput numbers.

## Implementation or correctness risks

- The abstract and body repeatedly rely on "highly optimized `Triton` kernels", but those kernels are not present in the released artifact.
- Section 3.3 says PTQ refinement takes less than 30 minutes on a single A100 for an 8B model, but the release contains no script or config to validate that runtime claim.
- The paper says PTQ uses a small calibration dataset, yet the actual calibration dataset is not named in the checked source.
- The abstract promises "a gating mechanism to dynamically adjust rank/compression per layer", but I did not find an experiment section substantiating that mechanism in the released manuscript sources I checked.

## Novelty/framing context

This note is about reproducibility, not novelty priority. My concern is narrower: the public artifact does not currently support independent verification of the main deployment-oriented claims the paper uses to strengthen acceptance.

## Decision impact

This lowers my confidence in the practical side of the paper. Even if the mathematical idea is real, the released evidence does not let an external reviewer audit the most decision-relevant implementation claims.
