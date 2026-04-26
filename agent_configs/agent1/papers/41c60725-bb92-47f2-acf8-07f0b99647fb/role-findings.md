## Reproducibility lead: central claim and reproduction target
Target: verify the claimed `1.79x-2.45x` simulation speedups and `2.06x-2.41x` real-world speedups for HeiSD, plus the accompanying generality claim for autoregressive VLA systems.

## Reproducer A: artifact-first check
- Koala tarball `41c60725-bb92-47f2-acf8-07f0b99647fb.tar.gz` is manuscript-only: TeX, figures, tables, and styles, but no code, configs, checkpoints, Qdrant dumps, task router, or normalization assets.
- The implementation section says the framework uses `5000 lines of code` (`_tex/6_implementation.tex`), but those files are not released.
- Appendix database details require `273,465` vectors across `40` task-specific collections with HNSW indexing and a `5.13 ms` query path (`_tex/10_appendix.tex`, `tab/apdx-DBdetails.tex`), yet none of the collections or builders are provided.

## Reproducer B: clean-room/specification check
- Online retrieval is defined as searching within the `pre-specified task collection` (`_tex/10_appendix.tex`), but the paper does not specify how task identity is chosen at inference time when claiming broad applicability.
- Real-world inference additionally depends on `task-specific statistics` loaded dynamically for action un-normalization after LoRA fine-tuning (`_tex/10_appendix.tex`), but those statistics and the loading logic are not specified or released.
- Algorithm 1 is malformed in the released source: line 25 reads `else B_t=True then`, so the online update branch is not executable as written (`alg/alg-1.tex`).

## Implementation auditor: code/artifact/repo match
- `github_urls` and `github_repo_url` are empty on Koala. There is no linked code artifact to match against the manuscript.
- The package contains no runnable implementation for the retrieval class, verify-skip logic, sequence-wise relaxed acceptance, or real-world fine-tuning path.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
- The strongest speedups rely on retrieval from smaller task-sharded search spaces and on hidden task routing. That makes the “good generality” claim materially narrower than the experiments section suggests.
- Real-world claims also depend on approximately `300 episodes per task type` for fine-tuning, so the deployment story is not just online acceleration; it includes substantial task-specific offline adaptation.

## Literature specialist: novelty/framing against permitted prior work
- I did not perform an external literature audit for this pass. The review focus here is reproducibility and implementation sufficiency.
