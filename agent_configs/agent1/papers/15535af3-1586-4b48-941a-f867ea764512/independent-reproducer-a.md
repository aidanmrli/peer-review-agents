# Independent Reproducer A Report

## Claim Tested

Could I run or reconstruct enough of DART to verify the paper's main speedup results from the released repository and linked artifacts?

## Procedure

I started from the author repository and paper sources. I inventoried files, checked for train/eval scripts, and examined the README-linked HuggingFace artifacts by HTTP metadata rather than downloading multi-GB files.

Commands:

```bash
git ls-files
du -ah . | sort -h | tail
sed -n '1,180p' README.md
rg -n "HumanEval|MT-Bench|LiveCodeBench|MBPP|Math500|CodeAlpaca|benchmark|eval|speedup|latency|tokens/s|train|gamma|anneal" .
curl -fsSLI https://huggingface.co/fvliang/dart-qwen3-ngram/resolve/main/full.trie
curl -fsSLI https://huggingface.co/fvliang/qwen8b-dart/resolve/main/model.safetensors
```

## Findings

The repository has a coherent inference/demo stack:

- `dart/model/dart_model.py`: DART model loading and generation.
- `dart/model/dart_utils.py`: tree decoding, posterior evaluation, inference input update.
- `dart/model/llama3_dart.py`: single-layer drafter with mask representations and shifted logits.
- `dart/tree_search/`: tree search, C++ trie integration, N-gram builder.
- `dart/app/`: Gradio demo and Qwen app launchers.

The linked Qwen DART weights and Qwen N-gram trie are reachable. The full trie metadata indicates a large artifact consistent with the paper's full-trie disk-size claim.

However, I found no scripts for the actual benchmark suite reported in the paper: MT-Bench, HumanEval, Alpaca, Math500, CodeAlpaca, LiveCodeBench, and MBPP. There is no task loader, timing protocol, prompt template bundle, baseline runner, result aggregation script, or saved output table. `main.py` is a single-prompt demonstration rather than a paper benchmark harness.

I also found no DART training code: no dataloader for ShareGPT/UltraChat, no annealed KL loss, no gamma sweep, no Flex-Attention training mask, no optimizer/schedule config, and no training entry point.

## Reproduction Outcome

Blocked for the central speedup claim. A reviewer could probably run Qwen DART inference after installing dependencies and downloading model/trie artifacts, but cannot recover the paper's reported speedup tables or training ablations from the release.

## Severity

High for acceptance because wall-clock speedup and training design are the core empirical claims.
