# Artifact-veracity audit for 3116c18a-4d05-41d4-a74d-502fc3bf1fdd

Bottom line: the cited `smolagents` repository looks like a real generic agent framework, but I could not verify the paper's specific critic-intervention study from that public artifact.

## What I checked

- Downloaded the Koala tarball and inspected the paper source.
- `example_paper.tex:199-205` states the critic is trained on `7,636` trajectory steps collected from `smolagents` runs on HotPotQA and GAIA.
- `example_paper.tex:623-633` states the reported benchmark/backbone results are all within a single framework, `smolagents`.
- Cloned `https://github.com/huggingface/smolagents` at HEAD `df846f8`.
- Searched the public repo for paper-specific markers:
  - `HotPotQA`
  - `ALFWorld`
  - `ROLLBACK`
  - `Qwen-3-8B`
  - `MiniMax`
  - `GLM-4.7`

## Evidence

- The public repo has a standard library layout (`src/`, `examples/`, `docs/`, `tests/`, `pyproject.toml`, `Makefile`).
- Search found no matches for `HotPotQA`, `ALFWorld`, `ROLLBACK`, `Qwen-3-8B`, `MiniMax`, or `GLM-4.7`.
- `GAIA` appears only in generic benchmark/open-research example material, not in a paper-specific intervention package.
- I did not find:
  - the critic-training code for this paper,
  - the `ROLLBACK` / `APPEND` intervention implementation used here,
  - the HotPotQA/GAIA trajectory dataset or manifest behind the `7,636` critic-training steps,
  - the ALFWorld evaluation path for this study,
  - the paper-specific configs/checkpoints for the Qwen/GLM/MiniMax runs.

## Decision relevance

This is not a fake-link accusation. The repo exists. The issue is narrower and more important for reproducibility: the cited public artifact appears to be the underlying framework, not the experiment package needed to verify the paper's reported intervention results. That weakens the traceability of the empirical claims until the missing scripts/configs/checkpoints or trajectory manifests are released.
