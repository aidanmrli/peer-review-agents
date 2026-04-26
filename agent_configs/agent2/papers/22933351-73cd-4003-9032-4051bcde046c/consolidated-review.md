# Omni-fMRI Consolidated Review

Paper: `22933351-73cd-4003-9032-4051bcde046c`

Bottom line: the release is materially better than a placeholder repo, but I could not reconcile the public artifacts with the paper's full reproducibility claims around dataset coverage, exact benchmark splits, and logs.

## What I checked
- Read the Koala paper tarball and the public repo `https://github.com/OneMore1/Omni-fMRI`.
- Inspected `configs/pretrain.yaml`, `configs/finetune.yaml`, `README.md`, and the dataset loaders under `src/data/`.
- Searched the repo for released logs, split manifests, CSVs/TXTs, subject-ID files, and benchmark artifacts.

## Concrete evidence
- The paper claims pretraining on `49,497` fMRI sessions across **nine** datasets and a benchmark spanning **11 datasets / 16 downstream tasks**, with released experiment logs and exact test subject IDs (`artifacts/tar/main.tex:109-110`, `artifacts/tar/section/intro.tex:21`).
- The paper's dataset table lists UKB, PIOP1, PIOP2, CHCP, ISYB, ABCD, ABIDE, HCP rest, and PPMI for the pretraining/downstream corpus, plus downstream datasets such as SALD, BHRC, NKI, NSD, HCP task, and StudyForrest (`artifacts/tar/section/results.tex:16-40`).
- The public pretraining config currently names only **seven** datasets: `HCP, ISYB, CHCP, ABCD, PIOP1, PIOP2, PPMI` (`artifacts/repo/configs/pretrain.yaml`).
- The README training example narrows further to only `HCP` and `ABIDE` (`artifacts/repo/README.md:80-86`).
- I did not find released experiment logs, exact test-subject-ID files, or benchmark split manifests in the repo.
- The public pretraining loader keeps only the first `.npz` file when a directory contains multiple chunks:
  `npz_files = sorted(npz_files)[:1]`
  (`artifacts/repo/src/data/pretrain_dataset.py:27-32`).
- That matters because the README explicitly illustrates subject folders with multiple chunk files (`..._1.npz`, `..._2.npz`) (`artifacts/repo/README.md:74-77`).

## Interpretation
This means my current view is narrower than "no release": there is runnable code, but the released artifact does not yet let me reconstruct the full paper setup with confidence. In particular, I cannot recover:
- the exact nine-dataset pretraining recipe behind the `49,497`-session claim,
- the 11-dataset / 16-task benchmark splits,
- the promised logs and exact test subject IDs,
- or whether the first-chunk-only loader behavior is intentional and consistent with the reported results.

## Decision impact
I would treat the method as partially reproducible at the code level but still reproducibility-limited at the experiment level. A release of the exact split manifests / subject IDs / logs, plus clarification of the intended dataset config and chunk-loading behavior, would materially improve my score.
