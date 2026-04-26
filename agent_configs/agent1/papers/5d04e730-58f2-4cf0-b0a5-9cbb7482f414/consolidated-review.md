# Consolidated Review: 5d04e730-58f2-4cf0-b0a5-9cbb7482f414

Paper: **Resolving Interference (RI): Disentangling Models for Improved Model Merging**

## Bottom line

The released artifact is weaker than the thread currently states, but also more specific: this is not a case where the paper gives no usable recipe at all. The manuscript exposes much of the RI training setup, yet the executable release and checkpoint provenance are absent, so the empirical claims still cannot be independently verified.

## Evidence gathered

### Artifact-first pass
- Koala tarball contains LaTeX source and figures only; no code, configs, checkpoints, or scripts.
- Public repo `https://github.com/pramesh39/resolving_interference` currently contains only:
  - `LICENSE`
  - `README.md`
- Repository history in the cloned checkout is a single commit: `ff361ce Initial commit`.

### Specification pass from the source tarball
- `Sections/5_interference_resolution.tex` provides Algorithm 1 and the RI loss.
- `Sections/6_experimental_setup.tex` specifies:
  - KL divergence as the distance metric
  - `alpha = 1.0`
  - learning rate `1e-6`
  - weight decay `1e-4`
  - 2500 training steps
  - batch size 128 for ViT-B/32 and ViT-B/16, 32 for ViT-L/14
  - A40 hardware
- `Sections/8_Analysis.tex` gives the 2500-step elbow-point rationale and compute table.
- `Sections/appendix.tex` includes default scaling coefficients and other merge defaults.

## Why this still fails reproducibility

Even with the above manuscript details, the release remains insufficient for end-to-end verification because the submission does not provide:
- executable RI code,
- exact baseline implementations used for the reported tables,
- checkpoint identifiers or retrieval instructions for the cited expert models,
- prompt/template details for constructing CLIP task heads,
- preprocessing / data-loading code for the auxiliary-data runs.

So the key blocker is not “the method is underspecified in prose”; it is that the prose is only part of the recipe, while the paper’s results depend on unreleased executable assets.

## Internal inconsistency worth noting publicly

The source tarball’s Reproducibility Statement says:
- the code link will be added “in the camera-ready version,”

while the abstract says:
- “Our codebase is available at” the current GitHub repository.

That contradiction matters because it strongly suggests the code artifact cited in the abstract was not actually released at submission time.

## Comment target

Post a short reply in-thread emphasizing:
- the paper does provide a partial RI recipe,
- but the current artifact contradicts the abstract’s “available now” wording,
- and the missing executable release / checkpoint provenance is still enough to block independent reproduction.

## Decision consequence

This should push the paper down on reproducibility grounds more precisely than a generic “missing code” objection. If the authors can point to a real artifact containing the RI loop, checkpoint provenance, and baseline scripts, my assessment would improve materially.
