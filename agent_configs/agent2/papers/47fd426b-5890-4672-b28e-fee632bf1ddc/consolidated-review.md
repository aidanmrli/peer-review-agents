# PAIR-Former: consolidated review evidence

Paper ID: `47fd426b-5890-4672-b28e-fee632bf1ddc`

## Bottom line

The BR-MIL framing is technically coherent and the manuscript is more explicit than average, but I could not independently reproduce the main empirical and runtime claims from the released Koala artifacts. The most important reasons are: the promised implementation/config release is missing from the tarball, and the reported evaluation uses a benchmark construction with disclosed overlap that limits how strongly I would interpret the near-saturated metrics.

## Evidence actually checked

I downloaded and extracted the Koala tarball:

```bash
mkdir -p papers/47fd426b-5890-4672-b28e-fee632bf1ddc/artifacts
curl -fsSL https://koala.science/storage/tarballs/47fd426b-5890-4672-b28e-fee632bf1ddc.tar.gz \
  -o papers/47fd426b-5890-4672-b28e-fee632bf1ddc/artifacts/source.tar.gz
tar -xzf papers/47fd426b-5890-4672-b28e-fee632bf1ddc/artifacts/source.tar.gz \
  -C papers/47fd426b-5890-4672-b28e-fee632bf1ddc
tar -tzf papers/47fd426b-5890-4672-b28e-fee632bf1ddc/artifacts/source.tar.gz | sed -n '1,200p'
```

Observed contents were limited to `preprint.tex`, figures, and style/bibliography files. No executable code, no config bundle, no checkpoints, and no supplementary implementation archive were present.

## Specific manuscript passages that drove my assessment

- The paper claims “An anonymized implementation is included in the supplementary material” (`preprint.tex:153`), but the delivered Koala tarball does not include it.
- The core pipeline is specific enough to understand at a high level: full-pool cheap scan, CPU STSelector, expensive re-encode on selected `K`, Set Transformer aggregation, and three-stage training (`preprint.tex:165-169`).
- Stage 3 is trained on a custom “released miRAWtest half-split,” with subsets `{1,2,3,4,5}` used for development and `{0,6,7,8,9}` held out for test (`preprint.tex:525-538`).
- The runtime appendix refers to released supplementary code/configs for CPU/thread counts, profiling knobs, and absolute tables (`preprint.tex:1947-1981`), but those materials are absent from the artifact package.
- The appendix discloses overlap between Stage 1-2 CTS data and Stage-3 test data: `68.9%` transcript overlap, `54.1%` miRNA-ID overlap, and `2.3%` exact pair overlap (`preprint.tex:2071-2080`).
- It also discloses overlap within Stage 3 itself: `60` positive train/test overlaps (`2.2%`) and `548/548` negative overlaps (`100%`) because of how `miRAWtest` is constructed (`preprint.tex:2084-2100`).

## Interpretation

Two-pass conclusion:

1. Artifact-first pass: not reproducible from released materials. The paper repeatedly points to supplementary implementation/configs that are not included in the Koala release, so I cannot audit the actual selector, training code, split-construction scripts, or runtime harness.
2. Spec-first pass: reproducible only at the conceptual level. The manuscript is detailed enough to follow the idea, but not enough to regenerate the exact results with confidence.

The overlap appendix is a useful disclosure, not a hidden flaw. Still, it materially changes how I read the headline metrics. The near-perfect precision/specificity numbers on the miRAW half-split are hard to interpret as strong evidence of broad pair-level generalization when Stage 3 train/test reuse all negative pairs and Stage 1-2 share many transcript/miRNA identities with the Stage-3 test set.

## Public-comment decision

My public comment should emphasize:

- the method idea looks plausible and sufficiently motivated;
- the current Koala artifact is manuscript-only despite contrary wording in the paper;
- the evaluation caveat is specifically the disclosed overlap/reuse structure, not an accusation of undisclosed leakage;
- a paper-specific implementation/config release plus explicit half-split construction scripts would materially raise my confidence.
