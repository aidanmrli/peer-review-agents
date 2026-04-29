# SurfelSoup: role findings

## Central claim and reproduction target
- Central claim: SurfelSoup achieves state-of-the-art point cloud geometry compression by combining probabilistic surfel primitives with an adaptive tree decision module under MPEG CTC evaluation.
- Reproduction target for this cycle: verify whether the paper exposes enough implementation detail and public artifacts to rerun the reported MPEG CTC D1/D2 results.

## Paper and artifact evidence checked
- Fetched and unpacked the Koala source tarball for `bacc72b6-2fca-4562-8698-195544579fc8`.
- Inspected `example_paper.tex` for training schedule, codec pipeline, evaluation dependencies, and release statements.
- Checked the tarball contents for code, scripts, configs, or weights beyond the manuscript sources.
- Read the current Koala thread to avoid duplicating the existing evaluation-framing comments.

## Reproducibility result from the smallest meaningful check
- The public release is manuscript-only: the tarball contains LaTeX sources, figures, and styles, but no executable code, configs, checkpoints, or evaluation scripts.
- The paper's reported rate points depend on a staged pipeline that is not reconstructible from the public artifact alone: five separate `lambda` models, a pretrain-then-finetune schedule, forced layer-1 surfel-node classification, G-PCC-Octree coding up to level `L=3`, and an extra super-resolution path for the lowest-rate points.
- The manuscript further states that full implementation, training/evaluation scripts, and weights will be released only upon acceptance.

## Implementation or correctness risks
- The strongest claims are benchmarked under MPEG CTC, but the executable mapping from manuscript to codec is missing.
- The lowest-rate results rely on a borrowed super-resolution stage, which materially affects the rate-distortion curve yet is not packaged in the present artifact.
- Because no code or weights are released, it is not possible to verify whether the adaptive tree decision, arithmetic coding path, and pSurfel reconstruction match the manuscript's reported operating points.

## Novelty/framing context
- The thread already covers ablation and generalization scope. My update is narrower: the practical reproducibility of the codec itself is currently unsupported by the public materials.

## Decision impact
- Positive: the paper may still contain a strong algorithmic idea.
- Negative: absent code, configs, and checkpoints, I would discount the empirical strength of the reported CTC gains and treat the submission as not independently reproducible in its current form.
