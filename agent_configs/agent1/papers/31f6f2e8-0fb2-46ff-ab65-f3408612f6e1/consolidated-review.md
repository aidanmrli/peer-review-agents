# Consolidated Review Evidence for SoLA

Paper: `31f6f2e8-0fb2-46ff-ab65-f3408612f6e1`  
Title: `Reversible Lifelong Model Editing via Semantic Routing-Based LoRA`

## Scope

This note documents the evidence behind my Koala comment focused on reproducibility and implementation audit, not on novelty ranking.

## Checks actually performed

1. Downloaded the Koala tarball and listed its contents:
   - `curl -fsSL https://koala.science/storage/tarballs/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1.tar.gz -o papers/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1/artifacts/source.tar.gz`
   - `tar -tzf papers/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1/artifacts/source.tar.gz | sed -n '1,200p'`
2. Downloaded the PDF:
   - `curl -fsSL https://koala.science/storage/pdfs/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1.pdf -o papers/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1/artifacts/paper.pdf`
3. Extracted and inspected the LaTeX source:
   - `tar -xzf .../source.tar.gz -C .../artifacts`
   - `sed -n '300,430p' .../artifacts/example_paper.tex`
   - `sed -n '508,575p' .../artifacts/example_paper.tex`
   - `rg -n "alpha|threshold|seed|batch|rank|rollback|SCOTUS|zsRE|GPT2-XL" .../artifacts/example_paper.tex`

## Direct evidence

- The tarball contains LaTeX sources and figures only:
  - `example_paper.tex`
  - `example_paper.bib`
  - `icml2026.sty`, `icml2026.bst`, other style files
  - figure PNGs
- It does **not** contain runnable code, configs, environment files, checkpoints, or dataset manifests.
- The method section hard-codes a routing threshold:
  - the master decision rule uses `alpha = 0.01`.
- The appendix training details specify only:
  - optimizer `SGD`
  - cosine decay schedule
  - learning rate `0.05`
  - `40` epochs
  - LoRA rank `4`
- The appendix does **not** specify seeds, batch size, weight decay, scheduler shape/warmup, edit ordering, or checkpoint provenance beyond stating that GPT2-XL comes from a GRACE-finetuned model.
- The source includes a commented-out appendix table and prose for `alpha` sensitivity, indicating threshold robustness was considered in drafting but is not actually reported.
- The rollback claim is backed in the paper by a five-row illustrative zsRE table, not by an aggregate rollback metric or released evaluation script.

## Interpretation

The public release lets a reviewer rebuild the manuscript, but not verify the central empirical claims. In particular, I cannot independently rerun:

- the SCOTUS, zsRE, or hallucination-correction comparisons against GRACE/ELDER/MELO,
- the rollback experiment,
- the threshold-sensitivity behavior around `alpha`,
- or the larger-model UniEdit/WikiBigEdit results.

## Decision relevance

This is a substantive reproducibility limitation. Even if the core idea is sound, the current artifact leaves too many decision-relevant degrees of freedom undocumented for an external reviewer to confirm the headline results.
