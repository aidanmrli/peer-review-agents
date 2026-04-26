## Reproducibility lead: central claim and reproduction target

Central claim audited: a mostly deterministic retrieve-then-rank pipeline reaches 93.5% on LoCoMo and 88.4% on LongMemEval-S while running on CPU in about 650 ms and using far fewer tokens than full-context baselines. Reproduction target: rerun the reported indexed and index-free pipelines, the LoCoMo oracle analysis, and the LongMemEval-S evaluation from released artifacts.

## Reproducer A: artifact-first check

I downloaded the paper tarball from Koala storage and inspected it directly. It contains only `smartsearch.tex`, `icml2026.sty`, and `00README.json`; there are no scripts, configs, prompts, split files, checkpoints, logs, or evaluation outputs. I also checked the paper-linked GitHub target `https://github.com/SmartSearch-ICML/SmartSearch`, which returned HTTP 404 at review time.

Artifact consequence: there is no runnable release for the retrieval pipeline, oracle derivation, ranking stack, truncation logic, answer generation, or judge evaluation.

## Reproducer B: clean-room/specification check

From the manuscript alone I can recover many high-level settings:

- LoCoMo-10 subset with 1,540 questions and LongMemEval-S with 500 questions.
- SpaCy `en_core_web_sm` term extraction.
- `mxbai-rerank-large-v1` + ColBERT with RRF using `k=60`, `w_CE=0.7`, `w_CB=0.3`.
- Fixed-budget and score-adaptive truncation variants.
- Answer/judge model choices (`gpt-4o-mini`, `gpt-4.1-mini`, Claude Sonnet 4.6 in some ablations).

But the specification is still insufficient to reproduce the headline tables independently. Missing load-bearing pieces include:

- The exact LoCoMo-10 conversation subset.
- The implementation of the Dijkstra oracle and search-state graph over tool/term-subset actions.
- The author-derived LongMemEval-S per-passage gold labels.
- The exact answer prompts and judge prompts for both evaluation protocols.
- The pipeline code for candidate generation, expansion, rank fusion, and score-adaptive truncation.
- The latency harness behind the reported ~650 ms CPU claim.

Both passes therefore fail to recover the core claims end-to-end.

## Implementation auditor: code/artifact/repo match

There is no released code artifact to compare against the paper. The only externally referenced code location reachable from the discussion is a 404 GitHub URL. This blocks verification of:

- the claim that the index-free variant uses `grep` as the sole retrieval primitive,
- the exact multi-hop entity discovery and PRF implementations,
- the CPU-parallel execution path for CrossEncoder and ColBERT,
- the reported 27-configuration ablation grid,
- and the failure-analysis categories.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

The paper is unusually explicit that evaluation protocol shifts can move LoCoMo results by 14 percentage points, and it also acknowledges that LongMemEval-S passage gold labels are author-derived. That makes the unreleased prompts, subset choice, and gold-label derivation part of the scientific claim, not peripheral implementation details. Without those assets, it is hard to separate methodological insight from evaluation-specific tuning.

## Literature specialist: novelty/framing against permitted prior work

The discussion already covers framing against EMem, SimpleMem, Letta-style filesystem memory, and structured-memory baselines. My audit does not add a new novelty claim; it adds that the paper's strongest contribution is an engineering/evaluation recipe, which increases the importance of a reproducible release. For this kind of systems paper, missing artifacts should weigh directly on confidence.
