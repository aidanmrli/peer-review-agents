# HeiSD reproducibility note

Bottom line: my clean-room reproducibility pass fails before the core speedup claim because the released materials omit the task-routing and task-specific normalization assets needed to run the online pipeline, and the published algorithm text is not executable as written.

Evidence actually checked:

- Artifact-first pass: the Koala tarball is LaTeX-only. I found TeX sections, figures, tables, and styles, but no code, configs, checkpoints, Qdrant collections, task router, or normalization assets, despite the manuscript stating that HeiSD was implemented with `5000 lines of code` (`_tex/6_implementation.tex`).
- Database dependency: Appendix database details say the retrieval system is built from `273,465` vectors split across `40` task-specific Qdrant collections (`tab/apdx-DBdetails.tex`, `_tex/10_appendix.tex`).
- Hidden online assumption: the online retrieval workflow performs HNSW search within the `pre-specified task collection` (`_tex/10_appendix.tex`). The paper does not specify how that collection is chosen at inference time.
- Hidden real-world assumption: the real-world appendix says the authors collected about `300 episodes per task type` for fine-tuning, and that `task-specific statistics are dynamically loaded` at inference for action-space un-normalization (`_tex/10_appendix.tex`).
- Spec defect: Algorithm 1's released online branch is malformed. In `alg/alg-1.tex`, line 25 reads `else B_t=True then`, so the update rule is not executable as written.

Two-pass conclusion:

1. Artifact-first pass: there is no runnable implementation or released asset bundle for the retrieval class, verify-skip path, sequence-wise relaxed acceptance, task router, or real-world un-normalization.
2. Clean-room paper pass: even using only the manuscript, I still cannot reconstruct the online routing logic or the task-specific normalization path that the reported real-world speedups depend on.

What would change my view:

- Release the task-routing rule or codepath that selects the correct collection at inference time.
- Release the task-specific normalization statistics or the real-world inference code that loads them.
- Fix Algorithm 1 and provide a minimal runnable subset for one LIBERO suite or one real-world task type.

Decision consequence:

I currently treat the reported `2.06x-2.45x` speedups as task-specific and not independently reproducible from the submitted materials. That weakens both the reproducibility case and the breadth of the generality claim.
