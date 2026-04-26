# Independent Reproducer B Report

## Claim Tested

Could I verify two operational claims independently: large-batch speedup behavior and "lossless" speculative sampling behavior?

## Procedure

I inspected the public generation paths and posterior-evaluation logic:

```bash
nl -ba dart/model/dart_model.py | sed -n '260,390p'
nl -ba dart/model/dart_utils.py | sed -n '220,285p'
rg -n "assert input_ids.shape\\[0\\] == 1|temperature|posterior|qx|batch|remain_total" dart/model dart/app
```

## Findings

The public generation path is batch-size-1 only. `DartModel.naive_generate` and `DartModel.dart_generate` assert `input_ids.shape[0] == 1`. This directly conflicts with the paper's reported batch-size experiments up to 64 unless there is an unreleased batch-capable runner. I found no alternate batch DART generation path or benchmark harness.

The greedy speculative path is relatively transparent: it compares candidate tokens against target argmaxes. The temperature-sampling branch is harder to reconcile with standard speculative sampling. In `evaluate_posterior`, the code sets `qx = 1.0` and appears not to use the draft proposal probability in the acceptance correction. Since the paper states DART preserves the target distribution, this path needs either proof, a test, or a separate implementation explanation. I did not find such a test in the repo.

## Reproduction Outcome

Contradicted or at least unsupported for large-batch results: the public code cannot reproduce them. Inconclusive but concerning for stochastic losslessness: the code path does not obviously implement a proposal-corrected acceptance ratio.

## Severity

High for large-batch claims; medium-high for temperature-1 "lossless" distribution claims.
