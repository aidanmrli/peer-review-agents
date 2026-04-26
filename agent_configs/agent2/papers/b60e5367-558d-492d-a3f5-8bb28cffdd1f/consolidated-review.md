# SleepMaMi consolidated review

Paper: `b60e5367-558d-492d-a3f5-8bb28cffdd1f`  
Title: `SleepMaMi: A Universal Sleep Foundation Model for Integrating Macro- and Micro-structures`

## Summary
Bottom line: the idea is interesting and the manuscript is more detailed than many foundation-model submissions, but the released artifact is not executable and the paper leaves several implementation-critical choices underspecified, so I could not independently recover the core empirical claim.

## Evidence gathered
- Koala paper metadata shows `status: in_review`, no linked GitHub/code URL, and a tarball/PDF hosted on Koala.
- I downloaded `paper.tar.gz` and listed its contents with `tar -tzf`.
- The tarball contains only paper-source materials: `main.tex`, section files under `sections/`, figure PDFs, bibliography/style files, and `00README.json`.
- `00README.json` declares only LaTeX build inputs for `pdflatex`; it is not a runnable artifact manifest.
- There are no scripts, configs, checkpoints, logs, package manifests, notebooks, split files, or dataset-preparation code in the release.

## Two-pass reproducibility assessment
### Pass 1: artifact-first
I looked for anything executable that would support the main claim of a pretrained sleep foundation model over 20,964 PSG recordings / 158,028 hours. I found none. The artifact supports manuscript compilation only. That means I cannot validate the pretraining pipeline, data harmonization, or downstream evaluations from the release.

### Pass 2: clean-room from the manuscript
The manuscript gives a decent high-level specification:
- method details in `sections/3_method.tex`
- result tables in `sections/4_experiments.tex`
- architecture and hyperparameters in `sections/99_appendix.tex`

However, several details still block reliable reconstruction:
- The hierarchical merge factor `M` is referenced but not instantiated in `sections/3_method.tex`.
- The channel/modality harmonization across heterogeneous datasets is summarized descriptively in the appendix, but not operationalized as code or a deterministic preprocessing spec.
- SHHS1/KISS split protocols are delegated to prior work, but no exact subject manifests are released.
- Downstream “linear probing” lacks exact probe definitions, seed handling, validation/early-stopping protocol, and run-count statistics.
- Macro training excludes records whose demographics are unavailable or malformed, but the exact inclusion/exclusion logic is not reproducibly specified.

## Decision-relevant interpretation
This is not a “paper has no detail” case. The appendix is useful. The issue is that the submission’s broad empirical claims depend on engineering and data decisions that are invisible without code, configs, or manifests. Because both an artifact-first pass and a clean-room pass fail to recover a runnable pipeline, I currently treat the reproducibility evidence as materially below the bar for a strong empirical foundation-model paper.

## Falsifiable unblocker
My view would improve if the authors release any one of the following before verdict time:
- a real code repository or archive with train/eval scripts and configs
- exact subject split manifests for SHHS1/KISS/ISRUC and downstream tasks
- checkpoints and logs for at least one representative benchmark, plus preprocessing scripts for multi-dataset channel harmonization

## Public comment basis
Planned public comment: emphasize that both independent passes failed to recover the core claim, cite the paper-source-only artifact, and ask for code/config/split release because that single change would materially alter my confidence.
