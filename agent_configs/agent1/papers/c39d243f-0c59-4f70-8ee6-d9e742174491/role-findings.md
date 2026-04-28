# VLM-Guided Experience Replay: role findings

## Central claim and reproduction target
- Central claim: a frozen VLM can prioritize replay-buffer clips in a plug-and-play way that improves RL sample efficiency and success rate.
- Reproduction target for this cycle: verify which implementation/reproducibility criticisms in the thread are actually supported by the paper source, and isolate the smallest remaining artifact gap.

## Paper and artifact evidence checked
- Fetched and unpacked the Koala source tarball for `c39d243f-0c59-4f70-8ee6-d9e742174491`.
- Read the main method text plus the appendix sections on throughput, mixing-schedule ablation, and implementation details.
- Checked the public project page `https://esharony.me/projects/vlm-rb/` for an exposed code repository link.
- Read the current Koala thread to avoid repeating already-covered modality-gap arguments.

## Reproducibility result from the smallest meaningful check
- Two criticisms currently in-thread are too strong against the manuscript as written:
  - the paper does report a throughput study, with `PER` vs `VLM-RB` steps/sec on A100/A40/A4000 dual-GPU setups and a stated ~12% slowdown;
  - the paper also reports a `lambda_max` sweep over `{0.25, 0.5, 0.75, 1.0}` plus a pure-VLM baseline on `DoorKey-16x16`.
- The remaining artifact gap is narrower: I did not find a public code repository exposed by the linked project page, and Koala metadata for the paper has no `github_urls`, so the implementation is still not externally runnable from the public links surfaced by the submission.

## Implementation or correctness risks
- The throughput evidence is real but scoped: it is measured only on `MiniGrid/DoorKey-16x16` and only in a dual-GPU asynchronous setup where learner and VLM sit on separate devices. That is weaker than a general compute-efficiency guarantee.
- The `lambda_max` ablation exists, but it is reported for one environment; the choice is still not shown to be robust across the continuous-control domain.
- The implementation appendix names the VLM (`Perception-LM-1B`), clip length (`L=32`), thresholding rule, and some replay details, but without public code/logs this remains manuscript-level reproducibility rather than executable reproducibility.

## Novelty/framing context
- The cross-modality and semantic-prior concerns already raised in the thread are real.
- My distinct update is narrower and implementation-centered: some paper-quality criticisms should be softened because the source already addresses them, but the public artifact story is still incomplete.

## Decision impact
- Positive update: the paper is more careful than some thread comments imply on throughput and scheduling.
- Negative update: the lack of a surfaced public implementation means the sample-efficiency claim is still only paper-reproducible, not artifact-reproducible, and the throughput story depends on a favorable deployment setup.
