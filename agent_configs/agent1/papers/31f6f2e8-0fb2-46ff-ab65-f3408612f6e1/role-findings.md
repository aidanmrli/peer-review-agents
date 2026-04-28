# SoLA Reproducibility Findings

## Central claim and reproduction target

The paper claims that SoLA is a reversible lifelong model-editing method that improves sequential editing performance over GRACE, ELDER, and MELO while enabling precise rollback by deleting a routing key. The smallest meaningful reproduction target for this review cycle was the public artifact itself: determine whether the release contains executable assets sufficient to recreate the reported SCOTUS, zsRE, SelfCheckGPT, UniEdit, and WikiBigEdit results, or at minimum the rollback demonstration and routing threshold behavior.

## Paper and artifact evidence checked

- Koala tarball: `papers/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1/artifacts/source.tar.gz`
- Tarball listing command actually run:
  - `tar -tzf .../source.tar.gz | sed -n '1,200p'`
- Result: tarball contains `example_paper.tex`, `.bib`, `.sty`, and figure assets only; no Python package, scripts, configs, checkpoints, dataset manifests, or environment files.
- Manuscript source inspected in `artifacts/example_paper.tex`.
- Key paper locations checked:
  - Main result table and rollback claim around the experiment section.
  - Routing threshold definition: `alpha = 0.01`.
  - Appendix training details.
  - Commented-out `alpha` ablation table and text in the appendix source.

## Reproducibility result from the smallest meaningful check

I could not reproduce any reported metric because the public release is manuscript-only. The tarball is sufficient to rebuild the PDF, but not to execute the method, rerun baselines, regenerate tables, or verify the rollback experiment. This means the current artifact does not support independent verification of the central empirical claims.

## Implementation or correctness risks

- No executable artifact: the release contains only LaTeX sources and figures, so there is no runnable SoLA implementation, no training script, no inference script, and no rollback utility.
- Missing backbone provenance: appendix training details state that GPT2-XL is obtained from the prior GRACE work rather than directly from an official release, but no checkpoint URL or exact commit/hash is provided.
- Missing training details: the appendix reports optimizer = SGD, learning rate = 0.05, epochs = 40, and LoRA rank = 4, but omits seeds, batch size, weight decay, cosine schedule parameters, early stopping behavior, and edit ordering protocol.
- Load-bearing routing threshold is under-specified: the method sets `alpha = 0.01` in the master decision rule, but the source also contains a commented-out appendix table and narrative for `alpha` sensitivity. That suggests threshold robustness was considered but is not actually reported in the paper.
- Rollback evidence is illustrative rather than reproducible: the rollback section presents only a five-row zsRE example table, and there is no script or aggregate evaluation artifact for rollback success.

## Novelty/framing context

This review did not use post-publication outcome signals. The novelty question is secondary here; the main issue is that the public artifact does not let an external reviewer audit whether the reported advantages over GRACE/ELDER/MELO arise from the method, the selected threshold, the chosen backbone checkpoints, or undocumented evaluation details.

## Decision impact

The method idea may still be promising, but the current release does not clear a basic reproducibility bar. This pushes my assessment downward unless the authors provide executable code, checkpoint provenance, dataset split/edit-order manifests, and enough configuration detail to rerun at least one benchmark plus the rollback evaluation.
