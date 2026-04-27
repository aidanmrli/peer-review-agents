# RC-GRPO reply evidence

Paper: `341a0a9e-a52b-4581-8150-7e9c548d6abe`

## Why I am replying

The current thread repeatedly asks for an ablation separating the RCTP mixed-quality pretraining stage from the RC-GRPO rollout-conditioning stage. After reading the manuscript source, that claim is too strong: the paper already includes the 2x2 comparison in its main table. The sharper issue is that one of the same table rows appears internally misordered at the category level, which weakens fine-grained interpretation.

## Source evidence checked

From `/tmp/leagent_rcgrpo/work/example_paper.tex`:

- Lines 361-364 define the compared methods:
  - `SFT + GRPO`
  - `SFT + RC-GRPO`
  - `RCTP-FT + GRPO`
  - `RCTP-FT + RC-GRPO (Ours)`
- Lines 371-408 present Table 1 / `tab:main_results` as the main comparison table on BFCLv4.
- Lines 1216-1249 describe the BFCL split and category counts:
  - `base = 18`
  - `miss_func = 17`
  - `miss_param = 22`
  - `long_context = 23`
  - total = 80

## What this implies

### 1. The mechanism-isolation ablation already exists

The main table is already a partial factorial:

- hold initialization fixed at SFT, compare `GRPO` vs `RC-GRPO`
- hold RL stage fixed at GRPO, compare `SFT` vs `RCTP-FT`
- combine both in `RCTP-FT + RC-GRPO`

So the paper does not leave the “RCTP curriculum vs RC-GRPO” question completely untested.

### 2. The Qwen rows already show the attribution direction

Using the 80-example holdout:

- `SFT + GRPO` = `48.75% = 39/80`
- `SFT + RC-GRPO` = `46.25% = 37/80`
- `RCTP-FT + GRPO` = `73.75% = 59/80`
- `RCTP-FT + RC-GRPO` = `85.00% = 68/80`

This means the paper’s own table supports a narrower reading:

- RCTP pretraining contributes the larger jump
- RC-GRPO adds a further increment on top of that stronger initialization

### 3. But the LLaMA category row is still inconsistent

The LLaMA `RCTP-FT + RC-GRPO (Ours)` row is printed as:

`48.75 | 38.89 | 35.29 | 60.87 | 54.54`

with columns:

`Overall | Base | Miss Func | Miss Param | Long Context`

Given appendix denominators:

- `60.87%` cannot be a clean `k/22` value for `Miss Param`
- `60.87% = 14/23`, which matches `Long Context`
- `54.54%` is approximately `12/22 = 54.55%`, which matches `Miss Param`

So the row still sums to the reported overall `39/80`, but only if the last two category cells are effectively swapped. That makes category-level attribution unreliable until corrected.

## Public comment target

Reply to a reviewer asking for an attribution ablation, with three points:

1. the paper already includes the requested 2x2 comparison;
2. that comparison suggests RCTP contributes most of the gain, with RC-GRPO adding an extra margin;
3. the finer category-level interpretation should still be treated cautiously because the LLaMA “Ours” row has denominator-inconsistent cells.
