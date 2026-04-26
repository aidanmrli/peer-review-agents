# Correctness Specialist Report

## Claim Tested

Whether the released code is consistent with the paper's algorithmic and losslessness claims.

## Findings

The basic architecture is coherent. A one-layer drafter consumes target hidden states and predicts multiple future logits in one pass. The shifted-logit design is plausible and visible in `dart/model/llama3_dart.py`.

However, several implementation details are decision-relevant:

- Batch claims are not supported by public code. `dart/model/dart_model.py` asserts batch size 1 in public generation functions, while the paper reports large-batch speedups.
- Tree-size semantics are ambiguous. The paper states `theta=59`; public defaults use `remain_total=60`.
- Tree scoring differs from the paper's formula. The paper describes log-softmax over logits, while the implementation applies softmax over the top-k subset before cross-depth scoring.
- The search extension disables N-gram scoring on the first token via a compile flag, which Algorithm 1 does not explicitly expose.
- For temperature sampling, `dart/model/dart_utils.py` appears to set the draft proposal term `qx` to 1.0 in acceptance logic. I did not find a distributional equivalence test, so the paper's broad "lossless target distribution" statement is insufficiently supported for stochastic decoding.

## Severity

Medium-high. These are not necessarily fatal to the core greedy demo, but they undermine the exactness of the paper-code mapping and the larger empirical claims.
