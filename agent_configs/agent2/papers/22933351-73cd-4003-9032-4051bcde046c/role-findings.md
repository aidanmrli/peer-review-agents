# Omni-fMRI Role Findings

## Reproducibility lead: central claim and reproduction target
Target: verify whether the public release supports the paper's core reproducibility claims around pretraining on 49,497 sessions across nine datasets and benchmarking across 11 datasets with released logs and exact test subject IDs.

## Reproducer A: artifact-first check
- Koala tarball is paper source only: `main.tex`, section tex files, figures, styles; no code, logs, split manifests, or subject-ID files in the tarball.
- Public repo exists and is non-empty: `pretrain.py`, `finetune.py`, configs, dataset code, Dockerfile, and Hugging Face link are present.
- I did not find released experiment logs, split files, CSV manifests, or exact test subject IDs in the repo. A `find` over `repo/` surfaced only code/logging utilities and no benchmark artifacts.

## Reproducer B: clean-room/specification check
- Paper claims: pretraining on 49,497 sessions across nine datasets and a benchmark spanning 11 datasets / 16 downstream tasks with released logs and exact test IDs (`tar/main.tex:109-110`, `tar/section/intro.tex:21`).
- The paper's dataset table includes UKB among the nine pretraining datasets and additional downstream datasets such as SALD, NKI, BHRC, NSD, HCP task, and StudyForrest (`tar/section/results.tex:16-40`).
- Current public pretraining config names only seven datasets: `["HCP", "ISYB", "CHCP", "ABCD", "PIOP1", "PIOP2", "PPMI"]` (`artifacts/repo/configs/pretrain.yaml`).
- README training example narrows further to `["HCP", "ABIDE"]` (`artifacts/repo/README.md:80-86`).

## Implementation auditor: code/artifact/repo match
- The core pretraining loader walks dataset folders and, when multiple `.npz` files are present in a directory, keeps only the first sorted file: `npz_files = sorted(npz_files)[:1]` (`artifacts/repo/src/data/pretrain_dataset.py:27-32`).
- README's own example shows a subject directory with multiple chunk files (`..._1.npz`, `..._2.npz`) (`artifacts/repo/README.md:74-77`), so this first-chunk-only behavior is directly relevant to the advertised setup.
- Finetuning code supports a few dataset modes and classes (generic CSV labels, HCP-task, ADNI), but the repo does not include the split manifests or label files needed to reproduce the 11-dataset benchmark as released.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
- The strongest concern is not that code is absent, but that the released code path may not instantiate the paper's benchmarked data regime.
- If first-chunk-only loading is the actual training behavior, effective pretraining data exposure may be materially smaller than a naive reading of the paper's session/chunk counts suggests.
- If it is not the intended training behavior, the release still lacks enough manifests/configs to recover the exact experimental setup.

## Literature specialist: novelty/framing against permitted prior work
- I did not perform a novelty comparison beyond the paper's own baseline framing. My comment should stay focused on release fidelity and reproducibility rather than novelty.
