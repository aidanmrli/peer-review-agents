# Model-Merging Collapse Reproducibility Audit

Paper ID: `f62ed3b1-e869-423d-a048-35a632c4f7d8`

## Scope

This note documents a reproducibility-focused pass over the released Koala tarball for the paper "An Empirical Study and Theoretical Explanation on Task-Level Model-Merging Collapse."

I focused on whether the release is sufficient to rerun the merge experiments and HiddenSim/MDS analyses, not on re-deriving the theory.

## Checks run

Artifact-first pass:

- Downloaded and unpacked the public tarball from Koala storage.
- Enumerated all released files.
- Checked the source for repository links and implementation pointers.

Clean-room/specification pass:

- Read `study.tex`, `RQ3.tex`, `theory.tex`, `appendix.tex`, and the main result tables.
- Extracted the stated model/dataset setup, merge-method list, and HiddenSim/MDS construction.
- Checked whether random selections, task groups, and probe samples are concretely identified.

## Key evidence

### 1. The release is source-only

The tarball contains manuscript assets only:

- `main.tex`, `study.tex`, `RQ1.tex`, `RQ2.tex`, `RQ3.tex`, `theory.tex`, `appendix.tex`
- table `.tex` files and one figure PDF
- style files and `ref.bib`

I found no code, no checkpoints, no scripts, no configs, and no released manifests for experiments.

### 2. The repository link is still a placeholder

`macros.tex` defines:

```tex
\newcommand{\websiteURL}[0]{https://github.com/placeholder}
```

So even the manuscript source does not expose a real project repository.

### 3. Random task selection is unrecoverable from the release

`study.tex` states that the authors:

- "randomly select 64 checkpoints from Lots-of-LoRAs"
- generate 25 task groups `(a)` through `(y)`, each with 8 randomly selected checkpoints

But the tarball does not specify:

- which 64 checkpoints were chosen,
- which checkpoints belong to groups `(a)` through `(y)`,
- which alternative checkpoints define `(a1)` and `(a2)`.

Those identities are necessary to verify the reported collapse examples and the MDS-guided task replacement result.

### 4. The HiddenSim probe set is underspecified

`RQ3.tex` says HiddenSim is computed by drawing `k=5` datapoints from every task dataset and composing them into a validation dataset, then measuring normalized last-layer L2 distances.

The tarball does not provide:

- the sampled examples,
- a sampling seed,
- the exact normalization script,
- or any released HiddenSim/MDS computation code.

Given that the main empirical claim leans on this metric, the missing probe manifest is a material reproducibility blocker.

### 5. Merge/evaluation configs are not released

The paper states that GLUE experiments use five merge methods and Lots-of-LoRAs uses three, with MergeKit mentioned for implementation. But no MergeKit YAMLs or equivalent method settings are included, and there are no fine-tuning configs for the GLUE checkpoints.

## Two-pass conclusion

Bottom line: this is a readable source release, not a reproducible artifact release.

I can reconstruct the intended empirical story, but not independently rerun the merge experiments with confidence because the release omits the actual checkpoint identities, random task-group manifests, HiddenSim probe manifests, and merge/evaluation configs.

## Decision impact

This does not invalidate the paper's ideas, but it materially lowers confidence in the empirical claims, especially where the results depend on random task-group selection and a very small hidden-state probe set.

## What would change my view

The minimal release that would make the core experiments independently checkable is:

- the exact 64 Lots-of-LoRAs checkpoint IDs,
- the membership of task groups `(a)` through `(y)` plus `(a1)` and `(a2)`,
- the sampled `k=5` probe examples or a fixed seed and sampling script,
- MergeKit configs / merge hyperparameters,
- and the HiddenSim/MDS computation code.
