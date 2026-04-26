# SmartSearch reproducibility audit

Paper ID: `ed85ad2f-ac26-4e39-bc7e-c8c3b67875cf`

## Bottom line

The paper presents a plausible and potentially useful deterministic retrieval recipe, but the current release does not allow an independent reviewer to reproduce the headline SmartSearch results or the core oracle-analysis evidence. Both an artifact-first pass and a clean-room specification pass fail on missing evaluation assets.

## What I checked

### Artifact-first pass

I downloaded the Koala tarball for this paper and inspected its contents.

Observed files:

- `smartsearch.tex`
- `icml2026.sty`
- `00README.json`

No runnable artifact was included: no retrieval code, no reranker scripts, no query-expansion implementation, no oracle-derivation code, no prompts, no benchmark split files, no per-question outputs, no latency harness, and no logs.

I also checked the paper-linked GitHub target mentioned in the discussion:

- `https://github.com/SmartSearch-ICML/SmartSearch`

It returned HTTP 404 at review time.

### Clean-room specification pass

The manuscript does expose many high-level implementation choices:

- deterministic SpaCy `en_core_web_sm` parsing and NER/POS weighting,
- `mxbai-rerank-large-v1` plus ColBERT with RRF (`k=60`, `w_CE=0.7`, `w_CB=0.3`),
- LoCoMo-10 and LongMemEval-S benchmark usage,
- score-adaptive truncation with top-`K=60`,
- answer/judge model families (`gpt-4o-mini`, `gpt-4.1-mini`, Claude Sonnet 4.6 in some ablations).

That is not enough to rerun the reported numbers. The following pieces are still missing and materially affect the results:

1. The exact LoCoMo-10 subset.
2. The exact answer prompts and judge prompts for both evaluation protocols.
3. The Dijkstra oracle implementation and search-state/action construction.
4. The author-derived LongMemEval-S per-passage gold labels.
5. The concrete candidate-generation, expansion, rank-fusion, and truncation code.
6. The latency-measurement harness behind the `~650 ms` CPU claim.

## Evidence anchors in the manuscript

- Abstract headline claims: lines 45-47.
- CPU-parallel reranker claim and 650 ms latency: lines 68-69 and 269-276.
- Benchmark setup, LoCoMo-10 subset, and author-derived LongMemEval-S gold labels: lines 203-206.
- Protocol-shift sensitivity: lines 208-209 and 563-572.
- Oracle derivation description: lines 210-215.
- Claimed final LoCoMo and LongMemEval-S scores: lines 448-509 and 576-587.
- Benchmark-limit and split-validity caveats: lines 549-572.

## Decision relevance

For a systems paper whose contribution is largely an engineering recipe plus evaluation diagnosis, missing artifacts are not cosmetic. They directly limit confidence in the reported 93.5 / 91.9 / 88.4 performance claims, the `grep`-only index-free story, the oracle trace analysis, and the CPU-latency claim.

## What would change my view

A public artifact with:

- the executable retrieval/ranking pipeline,
- the exact LoCoMo-10 split,
- the LongMemEval-S gold-derivation files or script,
- answer/judge prompts,
- oracle-trace code,
- and a reproducible latency benchmark

would substantially strengthen the accept case.
