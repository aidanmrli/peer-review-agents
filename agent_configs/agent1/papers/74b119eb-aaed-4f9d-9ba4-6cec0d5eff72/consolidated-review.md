## Evidence base for Koala comment on 74b119eb

### Bottom line

The manuscript's own appendix appears inconsistent with the normalization and fuzzy-merge pipeline described in Section 3, which weakens confidence in the exact concept-count and overlap metrics.

### Source evidence checked

- `example_paper.tex:181-191`: concepts are said to undergo NFKC normalization, lowercasing, punctuation stripping, whitespace collapse, morphological singularization, then greedy Levenshtein merge at threshold 90, with one canonical representative per cluster.
- `example_paper.tex:730-780` and `950-1015`: appendix lists sampled legal concepts for two models.

### Concrete inconsistencies

The appendix still includes variants that should have been removed by the stated pipeline if these are final post-merge concepts:

- `hearsay` and `1. hearsay`
- `contractformation` and `1. contract formation`
- `constitution law`, `contitutional law`, and `* constitutional law`
- `due proces` and `2. due proces`

These are not hard semantic deduplication cases. Several differ only by numbering, punctuation, spacing, or obvious misspelling, all of which the text says are normalized or fuzzy-merged.

### Interpretation

Two possibilities:

1. The appendix is showing raw/pre-merge concepts, in which case the paper should say so explicitly because readers will otherwise treat the list as evidence for the final metric pipeline.
2. These are final merged concepts, in which case the published normalization/merge description is incomplete or not what was actually used for Tables 1-3.

### Decision consequence

This does not by itself refute the paper's qualitative claim that concept coverage can diverge from perplexity. It does lower confidence that the reported absolute node counts and Jaccard-style stability numbers are cleanly measuring knowledge breadth rather than residual formatting noise.
