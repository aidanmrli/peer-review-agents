## Reproducibility lead: central claim and reproduction target

Target: reproduce the claimed `1.79x-2.45x` simulation speedup and `2.06x-2.41x` real-world speedup for HeiSD while preserving success rate on OpenVLA-based control.

Bottom line: I could not recover a runnable artifact from the submission package. The tarball is LaTeX source only (`00README.json`, `_tex/`, `tab/`, `fig/`, `alg/`) and does not include the claimed `5000 lines of code`, model configs, Qdrant collections, RLDS/HDF5 conversion scripts, or real-world fine-tuning assets.

## Reproducer A: artifact-first check

- Downloaded `/storage/tarballs/41c60725-bb92-47f2-acf8-07f0b99647fb.tar.gz` and listed contents. The package contains paper sources and figures only.
- The paper states "We use 5000 lines of code to implement the HeiSD framework" in `artifacts/_tex/6_implementation.tex:30`, but no code is released in the tarball and `get_paper` reports `github_urls: []`.
- The database is central to the method: 273,465 vectors, 40 task-specific collections, 6.5 GB total storage, Qdrant with `m=16`, `ef_construct=100` (`artifacts/_tex/10_appendix.tex:136-155`, `artifacts/tab/apdx-DBdetails.tex:1-17`). None of the collections, payload dumps, or builders are included.
- Real-world evaluation requires rebuilding the database and fine-tuning on approximately 300 episodes per task type (`artifacts/_tex/7_experiments.tex:45-52`, `artifacts/_tex/10_appendix.tex:326-347`), but those demonstrations and training scripts are absent.

## Reproducer B: clean-room/specification check

- Even with the paper text, the online retrieval contract is underspecified for reproduction. The system searches within a "pre-specified task collection" (`artifacts/_tex/10_appendix.tex:212-213`), but the paper does not specify how that collection is selected at runtime, or whether task identity is assumed known.
- The adaptive verify-skip algorithm is not executable as written. In `artifacts/alg/alg-1.tex:24-25`, the online update branch is malformed (`else B_t=True then`), and key quantities such as `T`, `Delta`, `S_c`, and `min(S_h)` are not operationally defined well enough to implement faithfully.
- The sequence-wise relaxed acceptance thresholds are reported as chosen "after multiple trials" with `bias_seq=30` and `bias_a_j=15` (`artifacts/_tex/4_rsd-optimization.tex:66-70`), but no search protocol, held-out split, or sensitivity table is provided for those safety-relevant settings.

## Implementation auditor: code/artifact/repo match

- Claim: end-to-end CPU+GPU implementation, Qdrant retrieval class, DeepSpeed-trained single-block drafter, OpenVLA-7B LoRA fine-tuning, Flash Attention 2 deployment (`artifacts/_tex/6_implementation.tex:22-41`, `artifacts/_tex/7_experiments.tex:8-14`, `artifacts/_tex/10_appendix.tex:339-351`).
- Release: no repository link, no code, no configuration files, no checkpoints, no database schema export beyond prose, no LoRA adapter weights, no prompt template, no environment setup.
- This mismatch is large enough that I cannot verify whether the measured speedups come from the proposed algorithm, system engineering choices, or task-specific implementation shortcuts.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

- The method explicitly does not analyze how verify-skip and relaxed acceptance alter the output distribution (`artifacts/_tex/7_experiments.tex:117-118`), so there is no formal guarantee that speedups preserve policy behavior beyond aggregate SR.
- Because the real-world results depend on unreleased data collection, adapter training, and task routing, the claimed minor SR loss is not independently checkable.

## Literature specialist: novelty/framing against permitted prior work

- I did not use external post-publication signals.
- Based on the submission only, the main decision-relevant issue is not novelty but reproducibility: the artifact gap prevents an independent check of the hybrid SD contribution.
