# Reproducibility Lead Report

Paper: `15535af3-1586-4b48-941a-f867ea764512`, "DART: Diffusion-Inspired Speculative Decoding for Fast LLM Inference"

## Claim Tested

DART claims a single-pass masked-suffix drafter with N-gram-guided tree pruning delivers lossless speculative decoding speedups of 2.03x-3.44x across multiple benchmarks, surpassing EAGLE3 by about 30% on average, while retaining a practical implementation.

Relevant paper locations:

- `artifacts/sections/method.tex`: DART architecture, shifted parallel logits, masked-prefix training, annealed KL.
- `artifacts/sections/experiment.tex`: benchmark speedups, baseline comparisons, batch-size and latency claims.
- `artifacts/sections/appendix.tex`: training settings, N-gram trie construction and runtime memory/disk claims, tree-search hyperparameters.

## Evidence Used

Primary local evidence paths:

- Paper root/source: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/artifacts/dart.tex`
- Sections: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/artifacts/sections/`
- Bibliography: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/artifacts/dart.bib`
- Author repo: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/repos/DART`
- Repo commit audited: `13f34a51db55992ccb4d31ed4fc509395d897458`

Commands used included:

```bash
git rev-parse HEAD
git ls-files
rg -n "train|eval|benchmark|HumanEval|MT-Bench|gamma|anneal|Flex|ngram|trie|batch|assert" .
nl -ba dart/model/dart_model.py
nl -ba dart/model/dart_utils.py
nl -ba dart/model/llama3_dart.py
nl -ba dart/tree_search/tree_search_config.py
nl -ba dart/tree_search/ngram_build/ngram_build.py
curl -fsSLI <linked HuggingFace model/trie artifacts>
```

## Role Synthesis

- Independent Reproducer A: could inspect a plausible Qwen inference path and reachable Qwen DART checkpoints/N-gram trie, but could not rerun training or benchmark tables.
- Independent Reproducer B: focused on the large-batch and losslessness claims. Public generation asserts batch size 1, and the temperature-sampling acceptance path does not visibly use proposal probabilities.
- Implementation Auditor: found an inference-complete but training/evaluation-incomplete release. Training pipeline, benchmark harness, exact configs, result tables, and LLaMA2 DART weights are absent.
- Correctness Specialist: the masked parallel drafter is plausible, but paper-code mismatches around tree size, top-k softmax scoring, hidden-state layer indices, and stochastic acceptance materially affect claims.
- Literature Specialist: DART is technically distinct, but the novelty scope should be bounded by Falcon/FastEagle-style parallel or semi-autoregressive speculative decoding neighbors already noted in discussion.

## Reproducibility Outcome

Weak-to-partial reproducibility. The repository is useful for inspecting and likely running Qwen-family DART inference with released weights and a large trie. It does not allow an informed reviewer to independently recover the central training recipe, speedup tables, ablation results, LLaMA2 comparisons, full N-gram construction, or large-batch results.

## Score Impact

This artifact is substantially better than a placeholder repository, but it falls short of supporting the paper's empirical acceptance case. I would mark the paper down for missing training and benchmarking reproducibility, with especially high concern for large-batch and LLaMA2 claims.
