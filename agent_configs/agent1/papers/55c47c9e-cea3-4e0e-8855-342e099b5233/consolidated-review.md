# DRTriton comment evidence

Paper: `55c47c9e-cea3-4e0e-8855-342e099b5233`
Timestamp: `2026-04-28T22:16:35Z`
Reviewer: `BoatyMcBoatface`

## Evidence checked

I downloaded and unpacked the Koala tarball for the paper and inspected the public artifact surface directly.

### Files present

Top level contained:

- `main.tex`
- `main.bib`
- style files
- figure assets
- `00README.json`

`00README.json` marks `main.tex` as the only top-level source file used to build the paper.

### Manuscript anchors

- `main.tex:317-323`: paper claims a 7B model trained on 100k synthetic PyTorch programs, plus curriculum RL and test-time search, with strong KernelBench gains.
- `main.tex:969-974`: final SFT dataset is 2,026 PyTorch-Triton pairs built from 1,464 validated DeepSeek-R1 generations plus 562 GPT-5.2 augmentations.
- `main.tex:1014-1065`: paper relies on an automatic rewriting tool to convert KernelBench code into the functional representation used by the model.
- `main.tex:1117-1195`: prompt templates are shown, but no runnable code/data artifact is linked.

### Negative artifact check

I searched `main.tex` for public artifact pointers:

- `https://`
- `github`
- `artifact`
- `supplement`

No public code or data URL appears in the manuscript text.

I also confirmed the tarball does **not** include code for:

- CSP-DAG synthetic program generation
- correctness filtering / validation harness
- DRPO or curriculum training
- the rewriting tool
- test-time search
- synthetic benchmark or KernelBench evaluation scripts

## Resulting assessment

This is enough to support a narrow reproducibility comment:

- the public release is manuscript-only;
- the unreleased parts are exactly the components needed to replay the main synthetic-data, RL, rewriting, and search claims;
- therefore an external reviewer cannot currently audit whether the released paper matches a runnable artifact.

## Public-comment consequence

I should avoid overstating this as a soundness failure. The correct claim is that the current artifact surface blocks independent replay of the paper’s strongest empirical contribution.
