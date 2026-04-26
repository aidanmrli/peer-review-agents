# Role Findings

Paper: `59386b0e-204c-4c09-986a-109be4967508`

## Reproducibility lead

Central claim and reproduction target: verify that the released artifacts support the paper's claimed `Graph-GRPO` contributions, namely analytic transition probabilities for RL training of graph flow models, GRPO-based optimization, and the refinement pipeline used for planar/tree generation and molecular optimization.

Bottom line: the public artifact trail does not currently support independent reproduction of the paper's core method. The strongest blocker is that the linked GitHub repository is the prior `DeFoG` project rather than a `Graph-GRPO` implementation.

## Reproducer A

Artifact-first check:

- Downloaded the paper tarball from `https://koala.science/storage/tarballs/59386b0e-204c-4c09-986a-109be4967508.tar.gz`.
- Listed bundle contents with `tar -tzf`; it contains `main.tex`, `ref.bib`, style files, and figures under `pic/`, but no Python, shell, notebook, checkpoint, config, or result files.
- This means the submission bundle is paper-source only, not a runnable reproduction package.

What was recovered:

- The tarball is sufficient to inspect the claimed method and tables.
- It is not sufficient to execute `Graph-GRPO`, recover planar/tree numbers, or reproduce molecular optimization.

## Reproducer B

Clean-room/specification check:

- Inspected `main.tex` from the source bundle.
- The paper repeatedly claims an RL method named `Graph-GRPO`, with analytic transitions, GRPO training, and a refinement loop for molecular optimization.
- The artifact pointer in the paper points to `https://github.com/manuelmlmadeira/DeFoG`.

Clean-room conclusion:

- The implementation described in the paper is specific enough that a dedicated code release should contain GRPO training code, reward/oracle handling, and refinement logic.
- The linked repository instead presents itself as `DeFoG: Discrete Flow Matching for Graph Generation`, i.e. a prior baseline, so the clean-room target and the released code pointer do not match.

## Implementation auditor

Code/artifact/repo match:

- Cloned `https://github.com/manuelmlmadeira/DeFoG` at commit `365bda9affadd5c2307014a0532ddaa244399441`.
- `README.md:1-4` identifies the repo as `DeFoG` and links the prior DeFoG paper, not `Graph-GRPO`.
- A repository-wide search for method-specific strings found dataset references like `planar` and `tree`, but no `Graph-GRPO` implementation markers. In particular, searching `README.md`, `src`, and `configs` for `GRPO|reinforcement|reward|oracle|dock|scaffold|valsartan|PMO|RL|refinement|perturb` returned only incidental uses of the word `refinement` inside graph-validity utilities and no GRPO/RL pipeline.
- The repo root contains the baseline training/sampling stack (`src/main.py`, `src/graph_discrete_flow_model.py`, configs, datasets) but no obvious Graph-GRPO-specific training entrypoint, molecular-reward code, or oracle budget machinery.

Assessment:

- The linked codebase appears to be relevant prior work and a plausible base model, but not the claimed method release.

## Correctness specialist

Methods/metrics/conclusion risks:

- Because the public code pointer appears to target the base model rather than the RL extension, it is not possible to verify whether the reported gains arise from the claimed analytic-transition GRPO training, from a different implementation, or from unreleased code.
- The paper says baseline results, including `DeFoG`, are cited from the original DeFoG paper, which makes the repository mismatch especially consequential: the public code may support the baseline but not the claimed improvement mechanism.

Score impact:

- This is a material reproducibility limitation for an empirical methods paper.

## Literature specialist

Novelty/framing against permitted prior work:

- The paper explicitly builds on `DeFoG` as the base graph flow model.
- Linking only the DeFoG repository without a Graph-GRPO extension risks collapsing the distinction between prior work and the claimed new method.
- That does not negate the paper's conceptual novelty, but it materially weakens the evidence that the new contribution is implemented and reproducible.
