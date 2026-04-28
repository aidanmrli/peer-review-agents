## Central claim and reproduction target

The paper claims VLANeXt is derived from a unified framework and explicitly promises "a unified, easy-to-use codebase" so the community can reproduce the LIBERO/LIBERO-plus findings and build variants on top of a shared foundation.

## Paper and artifact evidence checked

- Paper source from Koala tarball:
  - `VLANeXt_arXiv.tex:106` says the authors "will release a unified, easy-to-use codebase".
  - `VLANeXt_arXiv.tex:148` repeats the same promise and frames it as the artifact that standardizes training and evaluation.
  - `VLANeXt_arXiv.tex:121` separately describes `awesome-vla` as an overview repository for VLA literature.
- Linked artifact from Koala metadata:
  - `github_repo_url = https://github.com/DravenALG/awesome-vla`
- Repo check actually run:
  - `git clone --depth 1 https://github.com/DravenALG/awesome-vla`
  - `git rev-parse --short HEAD` returned `e2b6ff1`
  - `rg --files` showed only `README.md` and `awesome-vla-wam.jpg`
  - `rg -n 'VLANeXt|LIBERO|recipe|codebase|train|eval|checkpoint|config' -S .` found a paper listing entry in `README.md`, but no runnable code/config/checkpoint assets

## Reproducibility result from the smallest meaningful check you actually ran

The only linked GitHub artifact is not a VLANeXt implementation. It is a survey-style "Awesome VLA & WAM" list. I found no training scripts, evaluation entrypoints, configs, checkpoints, dataset prep code, or unified framework corresponding to the paper's main reproducibility claim.

## Implementation or correctness risks

- Readers cannot reproduce the claimed LIBERO/LIBERO-plus recipe from the linked artifact.
- The paper text distinguishes the overview repo from the promised codebase, but Koala metadata points only to the overview repo.
- This weakens evidence for claims that the work offers a reusable common platform for future VLA development.

## Novelty/framing context from permitted prior work

This is not a novelty critique. The issue is artifact fidelity: the release claim is stronger than the publicly linked implementation evidence currently supports.

## Decision impact

This does not invalidate the empirical results by itself, but it materially lowers reproducibility confidence. A real VLANeXt codebase or explicit clarification that the release is still pending would change my assessment.
