# Implementation Auditor Report

## Scope

I adversarially inspected whether the repository supports the paper claims, focusing on training, inference, evaluation/benchmarking, configs, checkpoints, N-gram trie artifacts, hard-coded paths, dependency pinning, and paper-code alignment.

## Repository Inventory

Author repo: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/repos/DART`

Audited commit: `13f34a51db55992ccb4d31ed4fc509395d897458`.

Present:

- Inference/demo: `main.py`, `dart/app/app.py`, `dart/app/qwen3_*_app.sh`.
- DART model implementation: `dart/model/dart_model.py`, `dart/model/dart_utils.py`, `dart/model/llama3_dart.py`, `dart/model/base.py`, `dart/model/kv_cache.py`, and modified target-model wrappers.
- N-gram/tree search: `dart/tree_search/tree_search.py`, `dart/tree_search/tree_search_config.py`, C++ sources under `dart/tree_search/cpplib/`, and `dart/tree_search/ngram_build/ngram_build.py`.
- Third-party EAGLE code under `third_party/eagle/`.
- Dependency metadata: `pyproject.toml`, `uv.lock`, `.python-version`.

Absent:

- DART training code.
- Paper benchmark harness.
- Exact training/eval configs.
- Result logs or raw tables.
- LLaMA2 DART checkpoint.
- Full N-gram corpus/build manifest.
- Large-batch generation implementation.

## Paper-Code Matches

The repository does implement a plausible DART inference architecture:

- `DartModel.from_pretrained` loads a base model, DART checkpoint, and trie.
- `dart/model/llama3_dart.py` implements a single-layer drafter with trainable mask embeddings and shifted parallel logits.
- Modified Qwen/LLaMA wrappers expose hidden states from selected layers.
- The C++ trie/search code and tree-search config implement top-k, beam width, N-gram weighting, logit weighting, and level weighting broadly consistent with Appendix hyperparameters.
- README links Qwen-family DART weights and the Qwen3 N-gram trie.

## High-Severity Gaps

1. Training code is absent. The paper's prefix-shared masked training, annealed KL, `gamma=0.6`, ShareGPT/UltraChat data, Flex-Attention sparse mask, optimizer, and schedule cannot be audited or rerun.

2. Benchmark code is absent. The paper reports MT-Bench, HumanEval, Alpaca, Math500, CodeAlpaca, LiveCodeBench, and MBPP speedups, but the repo has no benchmark loaders, prompt templates, timing harness, baseline commands, or aggregation scripts.

3. Public generation is batch-size 1 only. `dart_generate` and `naive_generate` assert batch size 1, while the paper reports batch sizes up to 64.

4. LLaMA2 paper results lack released DART weights or instructions. README lists Qwen3 DART weights only, while the paper reports LLaMA2-Chat-7B comparisons.

5. N-gram construction is only partially released. `ngram_build.py` is generic, but the Dolma 3 Mix version, filtering, sharding, preprocessing, node-count verification, memory benchmark, and Qwen full-build command are absent. `build_llama2.sh` uses hard-coded local paths such as `/data3/DART/ngram/train`.

## Medium-Severity Paper-Code Mismatches

- Paper final tree size is `theta=59`, while public defaults use `remain_total=60`; the root/sequence-count convention is not documented.
- Paper scoring uses log-softmax over position logits; code softmaxes only over top-k raw logits before tree expansion.
- The C++ search is compiled with `NO_NGRAM_ON_FIRST_TOKEN`, an implementation detail not obvious from Algorithm 1.
- Hidden-state layer selection is hard-coded and not transparently aligned with the appendix wording.
- Dependency pinning is brittle: `pyproject.toml` pins some packages but not all, and released HuggingFace configs report a different `transformers_version` than the local pin.

## Checkpoints and Artifacts

Qwen-family DART weights and Qwen trie files are reachable via README. I verified metadata only; no large artifacts were downloaded. No local checkpoints, datasets, generated tries, benchmark outputs, or training logs are included.

## Reimplementation Assessment

A reviewer can inspect and likely run Qwen DART inference with substantial setup. They cannot faithfully reimplement or reproduce training, benchmark tables, ablations, large-batch results, LLaMA2 comparisons, or full N-gram construction from the released materials.

Severity for acceptance: high.
