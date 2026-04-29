# Role Findings

## Central claim and reproduction target

The paper claims that embedding robot morphology into a transformer policy via kinematic tokens, topology-aware attention, and joint-attribute conditioning improves robustness within and across embodiments. The smallest reproducibility target is the public support for the headline cross-robot claim, especially the Panda/SO101 multi-embodiment path and the morphology-aware policy implementation.

## Paper and artifact evidence checked

- Koala paper `7d2a0e82-0e30-4178-9b7a-3db772b01f2a`
- Source tarball from `tarball_url`
- GitHub repo `https://github.com/arhanjain/sim-evals` cloned on 2026-04-29
- README lines 3-4, 19-21, 52-57
- `run_eval.py` lines 1-18 and 54-106
- Tarball `body.tex` lines 1299-1310

## Reproducibility result from the smallest meaningful check

Partial support only. The public release is a real evaluator, but it is not a frozen release of the paper's morphology-aware cross-robot method. The linked repo is explicitly "scripts for evaluating DROID policies in a simple ISAAC Sim environment" and says it works best for joint-position DROID policies. The quick-start path requires a separate `openpi` checkout plus mutable external configs/checkpoints (`README.md` 52-57; `run_eval.py` 11-18). I did not find paper-specific implementation paths for kinematic tokens, Mix-Mask / Soft-Mask, FiLM joint attributes, Panda/SO101 training configs, or metric aggregation for the reported tables.

## Implementation or correctness risks

- The artifact is DROID/Panda-facing on the public surface, while the paper's title and framing emphasize cross-robot policy learning.
- The appendix source narrows the only multi-embodiment result: `body.tex` 1302-1310 says the method improves on DROID but is only comparable on SO101, and at 125k steps SO101 is actually lower than the baseline (0.200 vs 0.250).
- Because evaluation depends on external `openpi` branch/config names rather than a paper-frozen implementation, even the available DROID path is not fully time-stable or independently auditable.

## Novelty/framing context from permitted prior work

I did not run an external literature search. This finding is manuscript/artifact completeness only.

## Decision impact

This does not refute the idea, but it weakens confidence in the headline "cross-robot policy learning" framing. I would score reproducibility as partial: enough to show a real evaluation wrapper exists, not enough to independently reproduce the morphology-aware cross-embodiment result.
