# APRIL artifact-veracity note

Paper: `3b91860c-3f48-4668-a978-5a403a2958eb`  
Title: `Learning to Repair Lean Proofs from Compiler Feedback`

## Bottom line

The APRIL release is real and useful, but the paper-to-artifact mapping is not clean for the flagship claims.

## Evidence checked

1. Paper source (`arxiv.tex`)
   - `109`: paper highlights a `260K` APRIL dataset and a headline `Qwen3-4B` repair result.
   - `269-275`: main evaluation centers the `Qwen3-4B` single-shot comparison against Goedel baselines.
   - `399-401`: explanation ablation centers the Qwen repair-only vs explanation-trained tradeoff.
   - `434`: reproducibility section points readers to `uw-math-ai/APRIL`, `uw-math-ai/gAPRIL-w-exp`, and `uw-math-ai/gAPRIL-wo-exp`.

2. Public artifacts checked on 2026-04-28
   - `https://huggingface.co/uw-math-ai/gAPRIL-w-exp`
   - `https://huggingface.co/uw-math-ai/gAPRIL-wo-exp`
   - `https://huggingface.co/datasets/uw-math-ai/APRIL`

## Findings

1. Both cited released model cards are explicitly `APRIL-Goedel-8B` models with base model `Goedel-Prover-V2-8B`. I did not find a public model card at the cited release URLs for the exact `Qwen3-4B` checkpoint behind the headline `27.4%` result.
2. The public APRIL dataset page currently shows a Hugging Face viewer/schema error indicating mismatching columns across files, so even simple schema inspection of the release is not frictionless.
3. The `gAPRIL-wo-exp` card says the model is repair-only and does not produce human-interpretable diagnostics, but its published usage template still asks for `Explanation`, `Fix`, and `Corrected Proof`. That weakens traceability for what the repair-only artifact is supposed to do.

## Decision consequence

This does not negate the core dataset contribution, but it lowers reproducibility confidence and means the paper should not imply that the current public release cleanly exposes the exact flagship Qwen result without additional clarification.
