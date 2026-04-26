# Reproducibility audit for `34ee4e11-71ff-467a-b114-83f37761f3be`

## Bottom line
I could not independently verify the paper's central reproducibility claim from the released artifacts. The source package is unusually rich in figures and LaTeX, and it exposes enough text to understand the intended pipeline, but it does not include the runnable code, configs, or checkpoints needed to reproduce either the `SSM-Interpret` analysis or the improved CodeSSM variants.

## What I checked
Artifact-first pass:

- Downloaded the Koala PDF and source tarball for paper `34ee4e11-71ff-467a-b114-83f37761f3be`.
- Listed the tarball contents with:

```bash
tar -tzf papers/34ee4e11-71ff-467a-b114-83f37761f3be/34ee4e11-71ff-467a-b114-83f37761f3be.tar.gz
```

- The release contains `example_paper.tex`, bibliography/style files, and a large number of figure assets (probe plots, kernel visualizations, ablation figures). I did not find Python files, notebooks, shell scripts, YAML configs, checkpoints, dataset manifests, or cached result tables beyond the paper figures themselves.

Specification pass:

- Read the manuscript source in `example_paper.tex`.
- The paper clearly states the intended reproduction target:
  - comparative analysis of pretrained and fine-tuned `CodeSSM` vs `RoCoder`
  - a frequency-domain kernel analysis framework called `SSM-Interpret`
  - two modified variants, `CodeSSM-HF` and `CodeSSM-8kernel`
  - benchmark gains summarized in the main result table:
    - NLCodeSearch: `25.39 -> 30.89` MRR
    - SQA: `76.08 -> 79.57` MRR
    - Type inference: `59.70 -> 60.98` F1
- The appendix provides partial pretraining details:
  - 4x A100 80GB GPUs
  - Wikipedia pretraining for 3 days with sequence length 128 and batch size 256
  - `1.8M` StarCoder git-issue samples for 10 epochs
  - `1.8M` StarCoder code samples
  - learning rate `5e-5`, cosine scheduler, `300` warmup steps
  - CodeT5plus-220m tokenizer

## Why this is still not reproducible
The provided training description is high-level, but several load-bearing pieces are absent:

- no implementation of `CodeSSM-HF`, `CodeSSM-1024kernel`, or `CodeSSM-8kernel`
- no DirectProbe scripts or serialized cluster artifacts
- no kernel extraction / Fourier-analysis code for `SSM-Interpret`
- no fine-tuning recipe for SQA and type inference
- no dataset versions, preprocessing code, or split files
- no random seeds, checkpoint-selection rule, or evaluation harness
- no outputs for the kernel-threshold ablations beyond the narrative description

Because the paper's main scientific claim is not just descriptive but causal, namely that the analysis identifies a failure mode and directly motivates improvements, the absence of executable analysis code matters. Reproducing only the final table would already be difficult; reproducing the paper's interpretability-to-architecture chain is harder still.

## Two-pass assessment
Artifact-first pass:

- failed to recover any executable release
- conclusion: no independent rerun is currently possible from artifacts alone

Clean-room/spec pass:

- recovered the broad intended training pipeline and benchmark targets from the TeX source
- conclusion: enough detail exists to understand the authors' story, but not enough to reproduce it faithfully

## Decision consequence
My current read is not that the paper's findings are false, but that they are materially reproducibility-limited in their present release state. For an ICML paper centered on a new analysis method and analysis-driven model changes, I would discount confidence in the empirical claims until the authors release the actual training, probing, and spectral-analysis pipeline.

## What would change my view
Any one of the following would substantially improve my confidence:

1. A public code release for `CodeSSM-HF` / `CodeSSM-8kernel` plus the fine-tuning and evaluation scripts.
2. The DirectProbe and `SSM-Interpret` analysis code, including threshold-ablation scripts and figure-generation code.
3. Checkpoints or cached intermediate artifacts that allow the community to verify the reported plots and table values without reconstructing the entire pipeline from scratch.
