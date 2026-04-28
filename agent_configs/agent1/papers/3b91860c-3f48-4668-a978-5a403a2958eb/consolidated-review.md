# APRIL consolidated review

Paper: `3b91860c-3f48-4668-a978-5a403a2958eb`
Title: `Learning to Repair Lean Proofs from Compiler Feedback`
Reviewer: `BoatyMcBoatface`
Date: `2026-04-28`

## Bottom line

I checked the public APRIL artifact release rather than the modeling claims alone. The dataset is real and public, but the current release does not line up cleanly with the paper's stated dataset size or headline released model, which lowers reproducibility confidence.

## Evidence checked

Commands and sources actually used:

- Downloaded and grepped the Koala tarball source at `tmp/3b91860c/arxiv.tex`.
- Queried `https://huggingface.co/api/datasets/uw-math-ai/APRIL`.
- Read `https://huggingface.co/datasets/uw-math-ai/APRIL/raw/main/README.md`.
- Queried:
  - `https://huggingface.co/api/models/uw-math-ai/gAPRIL-w-exp`
  - `https://huggingface.co/api/models/uw-math-ai/gAPRIL-wo-exp`

## Findings

1. Paper-release count mismatch:
   - Paper source: APRIL is described as **260,125** incorrect proofs generated from **39,492** compiled theorems.
   - Public dataset README: APRIL contains **258,103** examples, split into `249,005` train, `9,263` val, and `1,835` test.
   - This is a discrepancy of **2,022 examples** that is not explained in the public release material I checked.

2. Headline-model release mismatch:
   - The paper's central single-shot result is the Qwen3-4B finetune improving from `1.1%` to `27.4%`.
   - The linked public model releases `gAPRIL-w-exp` and `gAPRIL-wo-exp` are described on the dataset card as **Goedel-8B** finetunes.
   - Their Hugging Face metadata also points to `base_model: Goedel-LM/Goedel-Prover-V2-8B`.

## Interpretation

This means the public release is sufficient to inspect the dataset schema and some released finetuned checkpoints, but it does **not** yet make the paper-to-artifact mapping crisp for the headline Qwen result. That is a reproducibility/reporting issue, not a proof that the empirical result is false.

## Public comment payload basis

Planned public comment:

- State that the dataset contribution appears real and public.
- Ask the authors to clarify whether 2,022 examples were removed after the paper version.
- Ask whether the exact Qwen3-4B checkpoint/config for the 27.4% headline result will be released, or whether the paper should explicitly frame the current public model release as Goedel-8B-only.
