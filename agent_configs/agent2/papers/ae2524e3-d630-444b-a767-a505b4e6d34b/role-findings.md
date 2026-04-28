# Bird-SR reproducibility findings

## Central claim and reproduction target

Bird-SR claims a bidirectional reward-guided diffusion training recipe that improves real-world SR perceptual quality while preserving structure, with code available at `https://github.com/fanzh03/Bird-SR`. The smallest meaningful reproduction target is whether an independent reviewer can recover the load-bearing training specification for the reported DiT4SR/ResShift fine-tuning results from the public paper materials plus the linked repository.

## Paper and artifact evidence checked

- Submission tarball files:
  - `sec/3_method.tex`
  - `sec/4_experiment.tex`
  - `sec/X_suppl.tex`
  - `00README.json`
- Public repository:
  - `https://github.com/fanzh03/Bird-SR`

## Reproducibility result from the smallest meaningful check

Partial support only.

- The supplement does specify several training constants that are easy to miss from the main text:
  - `sec/X_suppl.tex:116-118` gives image resolution, learning rate `1e-6`, batch size `8`, and inference sampling schedules (`T=40` for DiT4SR, `T=15` for ResShift).
  - `sec/3_method.tex:30-36` and `sec/2_related.tex:19` state that real-LR reward optimization is applied only at the last reverse timestep.
- However, the public GitHub repo is still effectively empty for reproduction purposes:
  - clone on 2026-04-28 contained only `.gitignore`, `LICENSE`, and a one-line `README.md`.
- The paper materials I checked still do not expose the experiment-level pieces needed to regenerate table values:
  - total training duration / iteration count
  - dataset manifests or benchmark preprocessing scripts
  - exact checkpoint references
  - runnable configs / launch scripts

## Implementation or correctness risks

- Reviewers could reimplement a Bird-SR-like method from the equations, but exact replication of the reported numbers is not currently auditable.
- The abstract’s “code can be obtained” wording overstates current artifact availability.
- Because the executable release is empty, the paper’s reported gains remain dependent on author-side undisclosed engineering choices beyond the constants visible in the supplement.

## Novelty/framing context

This is narrower than “the method is unspecified.” The method is reasonably described at the equation level. The reproducibility failure is that the public release does not bridge the gap from method description to experiment replay.

## Decision impact

This moves me away from a pure soundness objection and toward a reproducibility/framing objection: the paper may describe a real method, but the current public evidence is not enough to independently verify the headline benchmark results.
