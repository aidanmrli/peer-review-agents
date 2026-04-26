# Consolidated Review Evidence

Paper: `15535af3-1586-4b48-941a-f867ea764512`, "DART: Diffusion-Inspired Speculative Decoding for Fast LLM Inference"

Agent: `agent1`

## Bottom Line

DART has a relevant and useful inference/demo repository, but it does not support independent reproduction of the paper's central empirical claims. The release is missing the training pipeline, benchmark harness, paper-equivalent configs, raw results, LLaMA2 DART weights, and public large-batch generation path. Several paper-code mismatches also affect exact speed/losslessness claims.

## Claim Being Tested

The paper claims DART obtains 2.03x-3.44x wall-clock speedups and beats EAGLE3 by about 30% on average via a single-pass masked drafter plus N-gram tree pruning, while preserving lossless speculative decoding behavior.

## Evidence Paths

- Paper: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/artifacts/dart.tex`
- Sections: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/artifacts/sections/`
- Repo: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/repos/DART`
- Repo commit: `13f34a51db55992ccb4d31ed4fc509395d897458`
- Existing implementation-auditor evidence file: `/tmp/agent1-4260-ledger/agent_configs/agent1/papers/15535af3-1586-4b48-941a-f867ea764512/implementation-auditor.md`

## Commands and Checks

```bash
git rev-parse HEAD
git ls-files
du -ah . | sort -h | tail
rg -n "train|eval|benchmark|HumanEval|MT-Bench|gamma|anneal|Flex|ngram|trie|batch|assert" .
nl -ba dart/model/dart_model.py
nl -ba dart/model/dart_utils.py
nl -ba dart/model/llama3_dart.py
nl -ba dart/tree_search/tree_search_config.py
nl -ba dart/tree_search/ngram_build/ngram_build.py
curl -fsSLI <linked HuggingFace model/trie artifacts>
```

I did not download multi-GB checkpoints or the full trie, and I did not run GPU inference because the local environment lacks the required ML stack.

## Role-by-Role Findings

### Reproducibility Lead

The artifact is partial. It supports architecture inspection and likely Qwen-family inference, but not independent recovery of training, benchmarks, ablations, LLaMA2 comparisons, full N-gram construction, or batch-size results.

### Independent Reproducer A

Could not reproduce the central speedup tables. The repo has no benchmark scripts for MT-Bench, HumanEval, Alpaca, Math500, CodeAlpaca, LiveCodeBench, or MBPP, and no timing protocol or result aggregation.

### Independent Reproducer B

Found public generation is batch-size-1 only despite paper results up to batch size 64. The stochastic acceptance path also needs clarification because proposal probabilities are not visible in the temperature branch.

### Implementation Auditor

The repo is inference-complete but training/evaluation-incomplete. Training code for the annealed KL/prefix-shared masked objective is absent. LLaMA2 DART weights are absent even though the paper reports LLaMA2-Chat-7B comparisons. N-gram build code exists, but the full Dolma 3 Mix construction manifest is not released.

### Correctness Specialist

The masked-suffix architecture is plausible. Paper-code mismatches remain around tree size, top-k softmax scoring, N-gram scoring on the first token, hidden-state layer indices, and stochastic losslessness.

### Literature Specialist

DART is distinct at the mechanism level, but the broader novelty framing should be tightened against close parallel-drafting neighbors already raised in discussion, especially Falcon/FastEagle-style approaches.

## GitHub Repository Status

Runnable code: likely for Qwen-family inference/demo after installing dependencies and downloading model/trie artifacts.

Training code: absent.

Evaluation scripts: absent for paper benchmark suite.

Configs: no paper-equivalent training/eval configs.

Checkpoints: Qwen DART checkpoints linked externally; no LLaMA2 DART checkpoint.

N-gram artifacts: Qwen trie linked externally; full construction pipeline incomplete.

Dependency pinning: partial and brittle.

## Reimplementation Assessment

A competent reviewer could rebuild a demo and inspect DART's inference mechanism. They could not faithfully reproduce the central empirical result from released materials. Missing details block training, ablation recovery, LLaMA2 reproduction, large-batch verification, and wall-clock benchmark reproduction.

## Literature References Used

Only permitted sources were used: paper artifacts, author repository, README-linked model/trie metadata, and prior-work framing visible in the paper and existing pre-verdict discussion. No OpenReview or future decision/status signals were used.

## Final Synthesis and Score Impact

The implementation evidence is mixed but decision-relevant. DART's released inference code raises confidence that the architecture exists, yet the missing training and benchmark artifacts are central for a fast-inference paper. I would give the artifact partial credit but substantially downgrade reproducibility confidence.
