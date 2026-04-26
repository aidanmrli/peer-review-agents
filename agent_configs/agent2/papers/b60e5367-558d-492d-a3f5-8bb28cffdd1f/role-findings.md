# SleepMaMi role findings

## Reproducibility lead: central claim and reproduction target
- Central claim: `SleepMaMi` is a universal sleep foundation model combining a micro encoder and a macro encoder, pretrained on 20,964 PSG recordings / 158,028 hours, and it outperforms prior foundation baselines on sleep staging, apnea segmentation, and some disease prediction tasks.
- Reproduction target: recover the pretraining pipeline and at least one downstream linear-probe result from the released artifact.
- Outcome: not reproducible from the released artifact in this submission state.

## Reproducer A: artifact-first check
- Artifact inspected: `papers/b60e5367-558d-492d-a3f5-8bb28cffdd1f/paper.tar.gz`.
- `tar -tzf` shows a paper-source tarball only: `main.tex`, section `.tex` files, figures, bibliography, and style files.
- No runnable code, no config files, no environment spec, no checkpoints, no data loaders, no train/eval scripts, no split manifests, and no preprocessing scripts were present.
- `00README.json` only declares LaTeX compilation inputs and `pdflatex`; it is not an executable artifact manifest.
- Artifact-first verdict: the release supports paper rendering only, not reproduction.

## Reproducer B: clean-room/specification check
- Method section gives useful high-level structure: micro encoder with private/shared transformers + MoE, macro encoder with bi-directional Mamba, MAE + contrastive loss, and DGCL over age/sex/BMI.
- Appendix gives some hyperparameters: batch sizes, training epochs, temperatures, learning rates, and a coarse architecture table.
- Critical missing details remain:
- The variable `M` for hierarchical patch merging is referenced in `sections/3_method.tex` but is not instantiated in the text.
- The exact channel-selection / modality-unification policy across datasets with heterogeneous sensors is not operationalized beyond a summary table.
- The SHHS1/KISS partitioning is delegated to prior work, but no subject-level split manifests are released.
- The downstream probe heads are described only as “linear probing” with loss/LR/batch size; no exact head definitions, early stopping, validation selection, or seed protocol are given.
- Macro training excludes some records when demographics are missing or malformed, but the exclusion rule is qualitative rather than machine-reproducible.
- Clean-room verdict: the paper is detailed enough to understand the idea, but not detailed enough to recreate the training/evaluation pipeline reliably.

## Implementation auditor: code/artifact/repo match
- Koala metadata lists no GitHub repository or external code URL for this paper.
- The tarball content matches a manuscript source dump, not a software artifact.
- The manuscript claims implementation in PyTorch / PyTorch Lightning and cites FlashAttention-2 in `sections/3_method.tex`, but no implementation files are released to audit against those claims.
- Result tables therefore cannot be checked against configs, logs, or checkpoints.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
- The reproducibility risk is the main correctness risk here: the empirical claims depend on a multi-dataset preprocessing pipeline and dual-stage pretraining recipe that cannot be independently executed.
- The paper acknowledges hardware distribution shift on KISS, which is useful, but without code/splits it is hard to determine how much of the reported gain depends on dataset-specific preprocessing and channel harmonization.
- The disease-prediction evaluation reports only C-index numbers without confidence intervals or repeated-seed variability, which weakens calibration for a clinical setting.

## Literature specialist: novelty/framing against permitted prior work
- The framing relative to prior sleep foundation models is plausible: the macro/micro split and demographic-guided macro supervision are reasonably distinct.
- My blocker is not novelty but evidence transfer: the submission does not provide an artifact that lets reviewers verify that the claimed engineering stack and benchmark protocol match the manuscript.

## Score impact
- Positive: interesting problem, sensible architecture, unusually detailed appendix tables.
- Negative: both reproduction passes fail on executable evidence.
- Decision impact: reproducibility evidence is currently below accept-level for an empirical foundation-model paper making broad benchmark claims.
