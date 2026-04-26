# Correctness Specialist Report

## Claim Tested

Whether the mathematical and algorithmic claims are internally coherent enough to support the compression interpretation, especially entropy-constrained one-vector coding and inference-time scaling.

## Checks

I checked the paper equations and the corresponding code paths:

- Flow-matching objective: `artifacts/chapters/method_zongyu.tex:22-35`; code at `video/train/stage2_vs/train_lora_single.py:412-426`.
- Entropy constraint: `artifacts/chapters/method_zongyu.tex:127-136`; code at `video/train/stage2_vs/unilora_utils.py:220-260`.
- Scaling algorithm: `artifacts/chapters/method_zongyu.tex:155-203`; code at `video/scaling/pipeline_wan_scaling_encode.py:646-701`.
- Compression/scaling hyperparameters: `artifacts/chapters/appendix.tex:585-603`.

## Findings

The basic flow-matching target is correctly reflected in code: the training target is `noise - latents`, consistent with the linear interpolant objective.

The vector-size arithmetic is plausible. For 81 x 832 x 480 frames, a 131072-element vector corresponds to:

- 0.004052 bpp at 1 bit/parameter.
- 0.012156 bpp at 3 bits/parameter.

This matches the qualitative bitrate regime shown in the paper's visual examples.

The scaling side-information arithmetic also matches the paper's stated range:

- 100 steps with `2^18` candidates require 1800 index bits, or 0.00005564 bpp.
- 1000 steps with `2^10` candidates require 10000 index bits, or 0.00030914 bpp.

However, the released implementation does not realize the paper's encoder-decoder contract. The algorithm text says the decoder should reproduce the chosen particle using the adaptation, shared PRNG, and transmitted index (`artifacts/chapters/method_zongyu.tex:161-166`). In the code, the target distribution is computed from `reference_latent` (`video/scaling/pipeline_wan_scaling_encode.py:646-647`), and the script obtains `reference_latent` from the original frames (`video/scaling/reconstruct_lora_single_scaling_encode.py:694-706`). It saves selected indices (`video/scaling/reconstruct_lora_single_scaling_encode.py:599-605`) but does not provide a decoder replay mode. This is a correctness failure for the implementation as a codec, not merely a usability problem.

There is also a paper-code mismatch in the stochastic selection rule. The paper says select an index from a categorical distribution proportional to importance weights (`artifacts/chapters/method_zongyu.tex:192-199`). The code adds Gumbel noise to log weights and uses `argmax` (`video/scaling/pipeline_wan_scaling_encode.py:684-692`), which is a valid Gumbel-max categorical sampling method in principle. But because the RNG for Gumbel noise is not explicitly tied to the same per-candidate shared PRNG stream as the Gaussian candidates, a separate decoder specification is needed to prove replayability.

## Severity

High. The math is not obviously invalid, but the released implementation falls short of a correct decompression demonstration for the inference-time scaling claim.
