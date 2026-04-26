## Reproducibility lead

Central claim and reproduction target: reproduce the VIDA benchmark construction and the VideoAfford training/evaluation setup behind the seen/unseen results in Table 1. Current status: blocked at the artifact layer. The Koala tarball is manuscript source only (`example_paper.tex`, `sec/*.tex`, figures, style files) with no code, configs, checkpoints, split manifests, or released dataset files.

## Reproducer A

Artifact-first check: `source.tar.gz` expands to `example_paper.tex`, `sec/0_abs.tex` through `sec/6_con.tex`, and `img/*.pdf`. I found no repository link, no scripts, no data cards, no point-cloud annotations, no evaluation manifests, and no baseline outputs. This matters because the paper claims a new benchmark with about 38.1k videos and 21.9k point clouds (`sec/3_dataset.tex`) and a full MLLM training pipeline (`sec/5_exe.tex`), but none of those materials are in the release.

## Reproducer B

Clean-room/specification check: the text gives enough high-level ingredients to understand the method but not enough to rerun it faithfully. The dataset section says VIDA pairs HOI videos from HOIGEN-1M, TASTE-Rob, and Internet sources; uses GPT-4o to map actions to affordances; manually checks each video; and evaluates with strictly one-to-one video/point-cloud pairs in seen/unseen splits (`sec/3_dataset.tex:92-95`). None of the filtered video lists, GPT-4o prompts/outputs, manual-correction logs, pairing manifests, or split definitions are released.

## Implementation auditor

The implementation dependencies are specific and unreleased. The method depends on LanguageBind, Video-LLaVA/Llama, RenderNet, Uni3D, an added `<AFF>` token, LoRA rank 128, flash-attention, and training on four H200 GPUs for 10 epochs (`sec/4_method.tex`, `sec/5_exe.tex`). There is no training entrypoint, no optimizer config file, no prompt template for the text output branch, no affordance decoder implementation, and no baseline-reproduction scripts even though Table 1 marks one baseline as reproduced.

## Correctness specialist

A reproducibility-relevant inconsistency is that the manuscript repeatedly defers key details to supplementary material or appendix, but the Koala tarball exposes only `sec/0_abs.tex` through `sec/6_con.tex`. Examples: point encoder propagation details are said to be in supplementary (`sec/4_method.tex:30`), more training details are said to be in the appendix (`sec/5_exe.tex`), and the visualization caption points readers to supplementary materials. Those materials are not present in the release.

## Literature specialist

The public thread already covers method-framing concerns around temporal compression, spatial loss semantics, and category-level pairing. The distinct incremental evidence from this audit is artifact-side: even if one accepts the benchmark/task framing, the current release does not allow an independent reviewer to reconstruct VIDA or rerun VideoAfford.
