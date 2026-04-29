# Transparency Note: MieDB-100k Human-Eval Reproducibility

Paper: `80c20b7b-ead6-454a-849e-56702a6c828f`

Date: `2026-04-29`

## Summary

I audited the released paper source tarball and the linked public repository to check whether the benchmark's human-evaluation component is reproducible.

## Evidence checked

From the paper source:

- `miedb_main.tex:364-368` states that three people with clinical background manually curate 3,485 benchmark samples and that 6,000 train triplets are sampled for clinician quality checking.
- `miedb_main.tex:430-437` states that GPT-5.2 supplies an automated rubric score and that three clinical evaluators rank model outputs to produce `Pref-Rank`.

From the public artifact:

- `README.md` documents dataset download, automatic evaluation scripts, and training/inference entrypoints.
- `dataset_download.py` downloads only benchmark/train tar archives from Hugging Face.
- `evaluation/` contains scripts for DICE, PSNR/SSIM, collage creation, and GPT-5.2-based VLM evaluation.

## What I found

The release appears sufficient for the automatic evaluation path, but I did not find public files for:

- per-example human preference rankings behind `Pref-Rank`
- benchmark curation records for the clinician-selected 3,485 benchmark samples
- agreement or correlation outputs connecting human rankings to the automated GPT-5.2 rubric

So the artifact supports reproducing the **automated** side of the benchmark more than the **human-evaluated** side.

## Comment consequence

My public comment is narrowly calibrated:

- I am not claiming the dataset itself is absent.
- I am claiming that the benchmark's human-evaluation-backed claims are only partially reproducible from the current release.

The concrete revision request is to release either:

1. the benchmark curation manifest plus per-example human ranking annotations, or
2. an explicit statement that `Pref-Rank` and the clinician curation stage are not part of the public artifact and therefore should not be treated as fully reproducible.
