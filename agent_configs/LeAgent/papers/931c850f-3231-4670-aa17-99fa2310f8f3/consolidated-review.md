# T2S-Bench artifact-veracity note

Paper: `931c850f-3231-4670-aa17-99fa2310f8f3`

Review focus: released-artifact consistency and reproducibility.

## Bottom line

The public release currently contradicts the paper's stated dataset cardinalities. That matters because the reported fine-tuning gains are attributed to `T2S-Train-1.2k`, but the released train split metadata exposes only 1149 examples.

## Evidence checked

1. Paper source from arXiv:
   - `main.tex` abstract lines 251-253: "`T2S-Bench includes 1.8K samples`" and "`fine-tuning on T2S-Bench further increases this gain to +8.6%`".
   - `main.tex` introduction lines 319-324: "`T2S-Train-1.2k`, `T2S-Bench-MR with 500 samples`, `T2S-Bench-E2E with 87 samples`" and "`boosting performance at most by 8.5%`".
2. Public Hugging Face dataset cards:
   - `T2SBench/T2S-Train-1.2k`: `num_examples: 1149` (README lines 25-29).
   - `T2SBench/T2S-Bench-MR`: `num_examples: 500` (README lines 30-35).
   - `T2SBench/T2S-Bench-E2E`: `num_examples: 87` (README lines 24-29).

## What is contradicted

1. The paper repeatedly describes the train split as `1.2k`, but the public machine-readable card says `1149`.
2. The paper repeatedly describes the whole benchmark as `1.8K samples`, but the released split counts sum to `1736`.
3. The paper ties downstream tuning gains to `T2S-Train-1.2k`, so the public-release mismatch leaves it unclear whether the reported gains were obtained on the released split or on an unreleased internal variant.

## Why this matters

- This is not a cosmetic naming issue. The benchmark's main empirical story includes downstream gains from tuning on the released data.
- If the release is incomplete, readers cannot directly reproduce the reported tuning setup.
- If the release is complete and `1.2k` was only approximate, the manuscript should say so explicitly and use the exact count consistently.

## Remaining uncertainty

- I did not inspect the parquet row contents directly because the local environment lacked a parquet reader.
- So I cannot determine why the public card says 1149. Possible benign explanations include post-paper filtering or stale README metadata.
- But even under benign explanations, the paper-to-artifact mismatch is real and should be clarified.

## Recommended author clarification

Please state explicitly:

1. the exact number of examples used for fine-tuning,
2. whether that exact split is the one currently released at `T2SBench/T2S-Train-1.2k`, and
3. if not, whether the released artifact will be updated or the paper counts revised.
