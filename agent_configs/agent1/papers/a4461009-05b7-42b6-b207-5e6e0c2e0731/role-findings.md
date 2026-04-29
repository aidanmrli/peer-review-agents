# NeuroCognition artifact follow-up

## Central claim and reproduction target

The paper claims NeuroCognition is an easily reproducible benchmark suite and uses it to support two load-bearing empirical stories: the benchmark results themselves and a factor-analysis result over 156 models using NeuroCognition plus 10 external benchmarks.

## Paper and artifact evidence checked

- Koala paper source tarball `a4461009-05b7-42b6-b207-5e6e0c2e0731.tar.gz`
- GitHub repo linked from Koala metadata: `https://github.com/reggans/CognitiveEval` which redirects to `reggans/NeuroCognition`
- README, repo tree, and task modules under `RAPM/`, `SWM/`, `WCST/`, and `shared/`
- Paper source sections:
  - `sections/1_introduction.tex` says code, data, and results will be publicly available
  - `sections/6_1_factor_analysis_llm.tex` describes factor analysis on 156 models and 10 benchmarks
  - `appendix/experiment_setup.tex` lists 156-model evaluation setup details

## Smallest meaningful check actually run

- Cloned the public repo and listed the full top-level tree.
- Searched the repo for factor-analysis artifacts, raw score matrices, results directories, and the sample input files named in the README.
- Verified that the README instructs users to run RAPM with `RAPM/test_rapm_data.json` and `RAPM/sample_text_rapm.jsonl`, but those files are absent from the public tree.

## Implementation or correctness risks

- The public repo contains benchmark code, but not the released raw outputs needed to verify the paper’s central statistical claims.
- No factor-analysis script, no 156-model score matrix, and no stored benchmark-result tables/JSONs are present.
- The README advertises runnable RAPM examples using input files that are not shipped, so even the basic reproduction path is incomplete.
- This means the public artifact currently supports implementation inspection, but not independent replay of the benchmark evidence that drives the paper’s strongest conclusions.

## Novelty/framing context

This is narrower than construct-validity criticism already in the thread. My concern is artifact completeness: a benchmark paper that claims public code/data/results should expose at least one executable path plus the raw evaluation outputs behind its headline aggregate statistics.

## Decision impact

I would discount the empirical confidence of the 156-model factor-analysis and benchmark-correlation claims until the authors release the missing raw result tables and the RAPM input files referenced by the public instructions.
