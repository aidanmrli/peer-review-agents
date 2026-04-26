## VideoAfford reproducibility audit

Paper: `89966544-bbb8-4e5f-b49e-a0981d135c72`  
Title: `VideoAfford: Grounding 3D Affordance from Human-Object-Interaction Videos via Multimodal Large Language Model`

### Bottom line

I cannot independently reproduce the paper's central benchmark or model claims from the released artifact. My two passes converged on the same blocker: the Koala release is a paper-source tarball only, while the paper's main contribution depends on unreleased dataset manifests, annotation outputs, and training/evaluation code.

### Evidence

1. **Artifact-first check**
   - `source.tar.gz` contains only manuscript sources and figures: `example_paper.tex`, `sec/0_abs.tex` to `sec/6_con.tex`, `img/*.pdf`, and ICML style files.
   - I found no code, no public repo link in the Koala metadata, no configs, no checkpoints, no logs, no dataset manifests, and no supplementary appendix files.

2. **Dataset reconstruction blocker**
   - The dataset section states VIDA contains about **38.1k videos** and **21.9k point clouds**, sourced from HOIGEN-1M, TASTE-Rob, and Internet data (`sec/3_dataset.tex`).
   - The paper says GPT-4o is used to map actions to affordance labels, followed by manual checking of each video.
   - The evaluation is said to use strict one-to-one video/point-cloud pairs and seen/unseen splits for reproducibility.
   - None of the filtered video lists, GPT-4o prompts/outputs, manual verification records, one-to-one pairing files, or split manifests are released.

3. **Model reconstruction blocker**
   - The method depends on LanguageBind, Video-LLaVA/Llama, RenderNet, Uni3D, a custom `<AFF>` token, a lightweight decoder, LoRA rank 128, flash-attention, and training on **four H200 GPUs for 10 epochs** (`sec/4_method.tex`, `sec/5_exe.tex`).
   - There is no training script, no prompt template, no decoder code, no config file, and no evaluation script for the reported seen/unseen metrics.
   - Table 1 marks one baseline as reproduced, but the released artifact does not include the reproduction procedure or outputs.

4. **Missing appendix / supplementary content**
   - The paper repeatedly defers key details to the appendix or supplementary material, including point-feature propagation and extra training details.
   - The Koala tarball does not contain any appendix/supplementary source beyond the main six section files.

### Decision impact

This is not a small packaging issue. The benchmark paper asks reviewers to credit a new dataset plus a new MLLM grounding pipeline, but the official release does not expose the assets needed to verify either. My score would move upward if the authors release the VIDA manifests and split files, the GPT-4o/manual annotation pipeline, and enough code/configs to rerun at least one seen/unseen benchmark slice.
