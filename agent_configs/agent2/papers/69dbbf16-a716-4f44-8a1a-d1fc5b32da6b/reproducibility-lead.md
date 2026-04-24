# Reproducibility Lead Synthesis

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`
Title: RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models
Role: Reproducibility Lead
Date: 2026-04-24

## Claims Tested

I coordinated the internal review around four decision-critical claims:

1. RoboAlign's SFT+RL training improves VLA performance on LIBERO, CALVIN, and real robot tasks, with headline gains of 17.5%, 18.9%, and 106.6%.
2. The RL stage uses less than 1% additional data and is the main driver of the performance gain.
3. The released EasyR1 and Isaac-GR00T artifacts are sufficient to reproduce the RL reward, data preparation, VLA conversion, and benchmark tables.
4. The novelty claim is that R1-style reasoning RL is directly aligned to low-level FAST action tokens and improves downstream GR00T-style VLA performance.

Minimum reproduction target: independently recover at least one main-table empirical result or execute a faithful code path for the paper-specific reward/training/evaluation pipeline. Tolerance before inspection: table arithmetic should match within rounding; empirical success rates require exact scripts/configs/checkpoints or raw logs sufficient to recompute values.

## Role Findings

Independent Reproducer A attempted a direct table and artifact reproduction. This role reproduced only arithmetic consistency: `12.8K / 2.28M = 0.56%`, CALVIN and real-world headline percentages match the base Qwen rows, and table averages are rounded consistently. The role could not reproduce the central empirical claim because no RoboAlign-specific reward implementation, SFT/RL training command, BridgeV2 FAST-token subset, checkpoint, VLA conversion code, or benchmark evaluation script was provided.

Independent Reproducer B used an independent consistency and code-trace route. This role reached the same conclusion without relying on Reproducer A: the table arithmetic is partially reconstructible, but the empirical claims are not reproducible. B also found denominator ambiguity: the LIBERO 17.5% headline matches `86.8` versus raw Qwen `73.9`, not the no-RL RoboAlign SFT row `78.7`; CALVIN and real-world headline denominators are likewise comparator-dependent.

The Implementation Auditor inspected the linked repositories at EasyR1 commit `dd71bbd252694f5f850213eec15795b6b88d9fea` and Isaac-GR00T commit `4b1dca9d88d2a0b9ea5a65aa61c82ff89f5c4f0e`. The audit found generic framework matches but no RoboAlign implementation: no FAST-token prefix-similarity reward file, no tokenizer/vocabulary extension code, no BridgeV2/DROID manifests, no SFT/RL launch configs, no Qwen2.5VL layer-18 to GR00T conversion patch, no CALVIN or real-robot evaluation pipeline, and no checkpoints or logs.

The Correctness Specialist identified major underspecification in the reward definition at `sections/method.tex:49-55`: if the first FAST token mismatches, the max set is empty; if the generated sequence is shorter, prefixes beyond its length are undefined; if it contains the target plus extra action tokens, the written reward still reaches 1.0. The role also found confounded ablations, a real-robot trial-count inconsistency, missing uncertainty estimates, and over-scoped representation and MLLM benchmark claims.

The Literature Specialist found a plausible but incremental contribution: applying GRPO/R1-style RL to low-level FAST-token accuracy is a useful target choice. The contribution is not a new VLA architecture, action representation, or RL method; it composes FAST/OpenVLA-style action tokenization, EasyR1/GRPO post-training, and GR00T-style VLA conversion. Prior embodied reasoning work such as ECoT, CoT-VLA, ThinkAct, Robot-R1, ManipLVM-R1, and RoboBrain makes the broader novelty framing too strong.

## Reproducibility Decision

Strong reproduction was not achieved. Both independent reproducers independently recovered table arithmetic but neither reproduced the central empirical claim. The implementation audit shows that this is not merely a local compute limitation: the released artifacts do not contain the paper-specific reward, data, checkpoints, configs, or evaluation scripts needed to rerun or recompute the main results.

Classification: weak reproducibility for the central empirical claim. The table values are internally consistent, but the empirical pipeline is unavailable and several correctness details are underspecified.

## Decision Impact

The public comment should foreground the reproducibility failure, while acknowledging that the reported table arithmetic is mostly internally consistent and the idea is literature-plausible. The acceptance case depends heavily on empirical improvements in robotics benchmarks; because those improvements cannot be independently reproduced from the supplied artifacts, confidence should be materially reduced.

Recommended verdict range if no additional artifacts are released: 3.5 to 4.5. This is a weak-reject range driven by reproducibility and correctness risk, not by lack of topical relevance. With a complete release and statistically supported matched comparisons, the paper could move upward because the research question is useful.

## Remaining Uncertainty

The authors may have private scripts and checkpoints that implement the stated method correctly. That uncertainty does not help the review, because the platform review standard is independent reproducibility from the paper, artifacts, and permitted prior work. Missing implementation details are therefore counted as a reproducibility limitation rather than neutral unknowns.
