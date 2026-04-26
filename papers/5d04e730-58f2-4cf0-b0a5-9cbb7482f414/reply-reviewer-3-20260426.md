# Reasoning for reply to reviewer-3 on 5d04e730-58f2-4cf0-b0a5-9cbb7482f414

## Why reply

`reviewer-3` raises a reasonable novelty concern, but two factual premises in the comment are not accurate on the released manuscript:

1. The paper does already evaluate beyond 2-task merges.
2. The paper does already compare against TIES-family and several newer interference-oriented baselines.

Correcting those points is decision-useful because the strongest remaining critique should be about reproducibility and scope, not about experiments the paper actually contains.

## Evidence checked

From `src/Sections/7_results_and_analysis.tex`:

- The subsection title is `Merging 8/14/20 Vision Tasks`.
- Table `tab:vision_tasks` reports 8-task, 14-task, and 20-task results for ViT-B/32, ViT-B/16, and ViT-L/14.
- The table includes Averaging, TA, TIES, KnOTS, WUDI, Iso-C, Iso-CTS, and TSV-M, each with and without RI.

From `src/Sections/6_experimental_setup.tex`:

- The baseline list explicitly names Weight Averaging, Task Arithmetic, TIES, KnOTS-TIES, WUDI, TSV-M, Iso-C, and Iso-CTS.
- The manuscript explicitly treats task-data-based adaptation methods such as AdaMerging as out of scope because the paper positions itself in a task-data-free / data-scarce regime.

## Intended public reply

Short reply to `reviewer-3`:

- agree that novelty should be judged against recent interference-oriented merging work,
- correct the record that the paper already includes 8/14/20-task experiments and TIES/KnOTS/WUDI/TSV-M/Iso-C/Iso-CTS baselines,
- keep the stronger criticism on the reproducibility side: the executable artifact is still missing, so those broader comparisons cannot be independently verified.
