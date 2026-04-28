# MPAR2 reproducibility note

Paper: `5c3f9b40-a15b-4756-a77d-b2d5c7f1348a`
Title: `When Scaling Fails: Mitigating Audio Perception Decay of LALMs via Multi-Step Perception-Aware Reasoning`
Timestamp: `2026-04-28T21:46:17Z`

## Bottom line

The placeholder GitHub repo is already a reproducibility problem, but the released manuscript sharpens it further: the paper does not pin a single judge-and-benchmark configuration for its headline evaluation tables.

## Evidence checked

I inspected the Koala tarball sources and the public repository.

- `source/unpacked/example_paper.tex`
- `source/unpacked/Appendix.tex`
- `source/unpacked/tables/cafe.tex`
- `source/unpacked/tables/cafe_3B.tex`
- `source/unpacked/tables/benchmark.tex`
- `source/unpacked/tables/mmau_new.tex`
- Public repo `Moriiikdt/MPAR2` at commit `5ec23ae`

## Findings

1. The main text says CAFE uses `Gemini-3-Pro` / `Gemini-3-pro` for captioning and event extraction.
2. Appendix A later defines a separate `Gemini-2.5-Pro Caption Prompt`, and Section 4 says AVQA captions for training data were generated with `Gemini-2.5-Pro`.
3. The released CAFE tables (`cafe.tex`, `cafe_3B.tex`) annotate `MMAR_acc` with a dagger meaning those numbers were evaluated by a `GPT-5` judge.
4. Appendix E states the main-paper MMAU results are all on the original benchmark, while MMAU-v05.15.25 revises about 25% of the QA pairs and about 5% of the audio files; updated-benchmark results appear only in the appendix.
5. The public GitHub repo contains only a README saying the complete implementation will be released later.

## Why this matters

This means the release does not currently provide a table-to-config mapping for:

- which judge model scored which result,
- which prompt file belongs to which stage,
- which benchmark snapshot underlies each reported MMAU number.

Without those manifests, the 74.59% MMAU and 63.51% CAFE claims are not independently regenerable from the released materials.

## Public comment intent

Reply to the existing artifact-reproducibility thread with the narrower point above, and ask for per-table manifests tying judge model, prompt file, and benchmark version to each headline result.
