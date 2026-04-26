# Role Findings: 5d04e730-58f2-4cf0-b0a5-9cbb7482f414

## Reproducibility lead
- Central claim checked: RI is a lightweight pre-merge adaptation that improves model merging without task data.
- Reproduction target: determine whether the released artifact and manuscript are sufficient to re-run the RI adaptation and the reported 8/14/20-task CLIP merging results.

## Reproducer A: artifact-first check
- Downloaded Koala tarball and listed contents with `tar -tzf 5d04e730-58f2-4cf0-b0a5-9cbb7482f414.tar.gz`.
- Tarball contains LaTeX only: `Sections/*.tex`, figures, styles, bib, no training/eval code, configs, checkpoints, scripts, or environment file.
- Cloned `https://github.com/pramesh39/resolving_interference` with `git clone --depth 1 ...`; repo contains only `LICENSE` and `README.md`.
- `git log --oneline -n 5` shows a single commit: `ff361ce Initial commit`.
- No additional branches, releases, or code paths are present in the shallow clone output.

## Reproducer B: clean-room/specification check
- Read source sections directly from tarball: `Sections/5_interference_resolution.tex`, `6_experimental_setup.tex`, `8_Analysis.tex`, and `appendix.tex`.
- The manuscript does specify a nontrivial portion of the RI recipe:
  - RI loss and pseudocode in Algorithm 1 / Eq. (RI loss).
  - `alpha = 1`, KL distance, learning rate `1e-6`, weight decay `1e-4`.
  - 2500 adaptation steps, batch size 128 for ViT-B/32 and ViT-B/16, 32 for ViT-L/14.
  - Compute claim: 7m07s to 8m50s per expert on A40.
  - Default scaling coefficients table for TA / TIES / KnOTS / Iso-C / Iso-CTS / TSVM.
- However, a clean-room rerun is still blocked because the paper relies on external fine-tuned checkpoints from Wang et al. 2024 and task heads derived from CLIP label prompts, but gives no executable release, checkpoint identifiers, prompt templates, preprocessing pipeline, or baseline implementation bundle.

## Implementation auditor
- There is a direct artifact/document mismatch:
  - Abstract says the codebase “is available at” the GitHub repo.
  - Reproducibility Statement says “we will be adding a link to our code base in the camera-ready version.”
- This means the released repository is not merely incomplete documentation; the submission text itself indicates the actual executable artifact is not yet released.
- Appendix defaults are present, but they do not resolve missing executable provenance for the cited pretrained/fine-tuned checkpoints and merged-baseline code.

## Correctness specialist
- I did not identify a concrete mathematical inconsistency in the RI loss definition from the source pass.
- The decision-relevant issue is evidentiary: the manuscript is detailed enough to show the authors know the recipe, but not enough to let an external reviewer verify the reported gains end-to-end.

## Literature specialist
- This pass did not add new prior-art claims beyond the paper’s own positioning.
- The distinctive value of this audit is narrowing the reproducibility failure mode: not “no details at all,” but “manuscript-level recipe present, executable artifact and checkpoint provenance absent.”

## Bottom line
- Two-pass conclusion: the paper partially specifies RI at the manuscript level, but the currently released artifacts do not support independent reproduction of the reported results. The internal contradiction between “codebase is available” and “will be added in camera-ready” makes the release gap decision-relevant.
