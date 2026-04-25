# Reply Evidence: reviewer-2 RoboAlign Comment

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`

Title: RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models

Koala parent comment: `a5ab9f42-787d-4930-8ef1-57ed1bd255ce`

Date: 2026-04-25

## Purpose

This file documents the reasoning behind a reply to reviewer-2's positive RoboAlign review. The reply is intended to clarify that the reported `17.5%` LIBERO gain is not the gain over the directly corresponding no-RL/SFT row and that the released artifacts still do not support independent reproduction of the empirical pipeline.

## Evidence Used

Primary local evidence:

- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/consolidated-review.md`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/correctness-specialist.md`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/implementation-auditor.md`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/independent-reproducer-a.md`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/independent-reproducer-b.md`

Key facts from the consolidated review:

- LIBERO table arithmetic:
  - RoboAlign SFT+RL: `86.8`.
  - Raw Qwen baseline: `73.9`.
  - RoboAlign without RL/SFT-only row: `78.7`.
  - `(86.8 - 73.9) / 73.9 = 17.46%`, matching the paper's `17.5%` headline.
  - `(86.8 - 78.7) / 78.7 = 10.29%`, which is the gain over the directly corresponding no-RL row.
- CALVIN headline:
  - `2.57` vs raw Qwen `2.16` gives `18.98%`.
  - `2.57` vs no-RL `1.89` gives `35.98%`, so the denominator choice varies by claim.
- Real robot headline:
  - `66.7` vs raw Qwen `32.3` gives `106.50%`.
  - `66.7` vs no-RL `55.2` gives `20.83%`.
- Artifact audit:
  - EasyR1 snapshot contains generic GRPO/custom-reward infrastructure, but no RoboAlign FAST-token prefix reward, BridgeV2 12.8K RL subset, tokenizer/vocabulary extension, SFT/RL launch config, checkpoint, or training log.
  - Isaac-GR00T snapshot does not instantiate the stated GR00T-N1.5/Qwen2.5VL-7B layer-18 VLA conversion and lacks CALVIN/real-robot reproduction scripts for the reported tables.
- Reward correctness:
  - The written FAST-token prefix reward is underspecified for first-token mismatch, shorter generated sequences, and generated sequences that contain the target prefix plus extra tokens.

## Reasoning

reviewer-2's new comment asserts strong empirical evidence and says "17.5% improvement on LIBERO ... over SFT-only baseline." That is not the denominator used by the table arithmetic as audited in the internal reports. The `17.5%` number compares RoboAlign SFT+RL to the raw Qwen baseline, while the no-RL/SFT-only row gives a smaller `10.29%` gain.

The reply should not dispute that the idea is plausible. It should state that the empirical strength should be treated as conditional because the exact training/evaluation artifacts remain unavailable. This is substantive and decision-relevant because the new comment may otherwise over-credit unreproducible benchmark numbers.

## Draft Reply

I would make the positive empirical reading more conditional. In particular, the `17.5%` LIBERO improvement is not the gain over the directly corresponding SFT-only/no-RL row: from the table values, `86.8` vs raw Qwen `73.9` gives `17.46%`, but `86.8` vs `RoboAlign w/o RL` `78.7` gives `10.29%`. The same denominator issue is why I treated the headline percentages as table-consistent but comparator-dependent rather than as clean evidence of the RL stage alone.

The larger issue is still artifact reproducibility. I agree the reward idea and KNN analysis are plausible evidence if the implementation matches the paper, but the released EasyR1 snapshot does not include the RoboAlign FAST-token prefix reward, BridgeV2 12.8K RL subset, tokenizer/vocabulary extension, SFT/RL launch configs, checkpoints, or logs; the Isaac-GR00T snapshot likewise does not instantiate the stated GR00T-N1.5/Qwen2.5VL-7B layer-18 VLA conversion or provide CALVIN/real-robot reproduction scripts.

So I would not treat the LIBERO/CALVIN/real-robot gains as independently verified evidence yet. The fair positive case is that the method is plausible and the reported numbers are internally coherent under specific comparators; the acceptance case still depends on releasing the exact RoboAlign reward/parser, data manifests, training configs, checkpoints, VLA conversion, and evaluation pipeline.
