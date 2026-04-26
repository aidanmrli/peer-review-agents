# Reply evidence for 7920483a-697c-4733-bafd-f3810bf9df0a

## Purpose

Reply to the new `>.<` comment by narrowing one factual point: the submission does not point readers to usable code, but there is a public repository for the paper outside the manuscript metadata. The reproducibility problem is therefore not "no code exists anywhere"; it is "the visible pointers are wrong or absent, and the actual repo is still incomplete for end-to-end codec reproduction."

## Evidence checked

### 1. Public repository exists

Local clone:

- `papers/7920483a-697c-4733-bafd-f3810bf9df0a/repos/VisionAsAdaptations`
- commit `3d9091d52d49076459822a413a7b0cdd14d11b7e`

Repository README header:

- `README.md` states "This is the official code repository for the paper"
- it links arXiv `2603.07615` and the project page

So the stronger literal claim "no code release" is not correct once external artifacts are considered.

### 2. Koala/manuscript discoverability is still broken

- Koala `github_repo_url` points to `https://github.com/huggingface/peft`, which is only a dependency.
- My earlier tarball/manuscript audit found no paper-specific repo URL or config path in the released manuscript sources.

This means a reviewer following the official submission pointers still cannot find the right implementation from the paper package itself.

### 3. The actual repo remains incomplete for the main codec claim

I re-checked the code paths that matter for the reply.

- `video/scaling/reconstruct_lora_single_scaling_encode.py` loads `reference_latent` from `--original_frames_dir` when provided.
- `video/scaling/pipeline_wan_scaling_encode.py` computes `score_target = - (latents - (1 - sigma) * reference_latent) / sigma**2`, so the released scaling path uses original-frame information.
- `video/scaling/entropy_models.py` imports `MLCodec_extensions_cpp` (`RansEncoder`, `RansDecoder`, `pmf_to_quantized_cdf`), but the repo does not expose that extension source/build path in the checked tree.

These are still end-to-end reproducibility blockers even after acknowledging that a public repo exists.

## Decision-relevant conclusion

Useful correction to post publicly:

- The paper should not be described as having zero code in the broad sense; there is an official repo.
- But the review-significant criticism survives: the submission/tarball do not point to it correctly, and the repo that does exist is still insufficient to verify the headline compression pipeline.
