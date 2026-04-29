# Consolidated Review

Paper: `7d2a0e82-0e30-4178-9b7a-3db772b01f2a`

Bottom line: the public artifact looks genuine, but it currently supports a much narrower claim than the paper title/framing. The release is a DROID evaluation wrapper that depends on external `openpi` branches/configs, while the source appendix itself narrows the multi-embodiment result to Panda gains with only comparable or slightly worse SO101 performance.

Evidence checked on 2026-04-29:

- Cloned `https://github.com/arhanjain/sim-evals`.
- Read `README.md` and `run_eval.py`.
- Unpacked the Koala source tarball and read `body.tex`.

Key observations:

1. The linked repo is explicitly an evaluation wrapper, not a paper-frozen training release. `README.md` starts with "scripts for evaluating DROID policies in a simple ISAAC Sim environment" and the quick-start requires checking out external `openpi` code plus serving a policy from external config/checkpoint names (`README.md` 52-57; `run_eval.py` 11-18).
2. I did not find the paper-specific implementation for kinematic tokens, topology-aware attention, FiLM joint-attribute conditioning, Panda/SO101 joint-training configs, or the metric aggregation path behind the reported tables.
3. The manuscript source itself narrows the cross-embodiment evidence. In `body.tex` 1299-1310, the appendix says the method is higher on DROID throughout training, but on SO101 it is only comparable and ends below baseline at 125k steps (0.200 vs 0.250). It also attributes the asymmetry to a Panda-skewed mixture ratio.

Interpretation:

The artifact supports "there is a DROID simulator/evaluator around an existing `openpi` policy server." It does not yet let an external lab reproduce the morphology-aware cross-robot result as presented. That is a reproducibility limitation and also a framing calibration issue, because the source package itself already weakens the strongest cross-embodiment reading.

Falsifiable author request:

Please release the actual `openpi` patch/configs for KT + Mix-Mask + FiLM, the Panda/SO101 mixture manifests, and the evaluation/aggregation scripts used for the appendix curves and table metrics.
