# Independent Reproducer B Report

## Claim Tested

Could I verify the compression-specific contract of VOV: the encoder transmits the adaptation vector plus small side information, and the decoder reconstructs the video without access to the original frames?

## Procedure

I independently inspected the scaling, entropy, and reconstruction paths rather than starting from the README examples alone.

Commands:

```bash
nl -ba repos/VisionAsAdaptations/video/scaling/reconstruct_lora_single_scaling_encode.py | sed -n '560,625p'
nl -ba repos/VisionAsAdaptations/video/scaling/reconstruct_lora_single_scaling_encode.py | sed -n '650,715p'
nl -ba repos/VisionAsAdaptations/video/scaling/pipeline_wan_scaling_encode.py | sed -n '600,735p'
rg -n "best_idx|saved_idx|reference_latent|pickle.load|load.*best_idx|compress\(|decompress\(|EntropyCoder|bitstream" repos/VisionAsAdaptations/video repos/VisionAsAdaptations/image
awk 'BEGIN{pixels=81*832*480; printf("100x2^18 side bits: %d, bpp: %.8f\n",100*18,(100*18)/pixels); printf("1000x2^10 side bits: %d, bpp: %.8f\n",1000*10,(1000*10)/pixels); printf("vector 131072 at 1/3 bits-param bpp: %.6f / %.6f\n",131072/pixels,3*131072/pixels)}'
```

## Findings

The released scaling reconstruction script uses original frames during reconstruction. `video/scaling/reconstruct_lora_single_scaling_encode.py:694-706` loads `reference_latent` from `--original_frames_dir`. The pipeline then computes the target score from that reference in `video/scaling/pipeline_wan_scaling_encode.py:646-647`.

The code saves `best_idx` at `video/scaling/reconstruct_lora_single_scaling_encode.py:599-605`, but I found no complementary decoder script that loads these indices and replays the selected candidate path without `reference_latent`. `rg` finds saving of `best_idx` but no `pickle.load` or command-line argument to consume it. This is not equivalent to the paper's claim in `artifacts/chapters/method_zongyu.tex:161-166` that the decoder reproduces the selected particles using the adaptation and shared PRNG.

The released scaling implementation hard-codes `candidates_num = 2**10` at `video/scaling/pipeline_wan_scaling_encode.py:667-668`; the paper also reports a 100-step, `2^18` candidates-per-step setting (`artifacts/chapters/experiments.tex:138-139`, `artifacts/chapters/appendix.tex:602-603`). I found no script flag exposing `2^18`.

The side-information arithmetic is plausible but nonzero:

- 100 steps x `log2(2^18)` = 1800 bits = 0.00005564 bpp for 81x832x480.
- 1000 steps x `log2(2^10)` = 10000 bits = 0.00030914 bpp for 81x832x480.
- The base vector at 131072 parameters is 0.004052 bpp at 1 bit/parameter and 0.012156 bpp at 3 bits/parameter.

The repo has an entropy model and rate estimator, but I found no end-to-end arithmetic/rANS or other bitstream writer invoked for the released VOV vector. `video/train/stage2_vs/unilora_utils.py:249-260` estimates rate by probability under a factorized model, and `video/train/stage2_vs/entropy_models.py` defines coder classes, but the searched code has no complete command that encodes and decodes the vector bitstream used for plotted RD points.

## Reproduction Outcome

Contradicted for decoder-level scaling reproducibility. The released code can perform an encoder-side selection experiment with the original latent available, but it does not demonstrate that the saved indices plus vector are sufficient for a separate decoder to reproduce the scaled reconstruction.

## Severity

High. This directly affects the "compression" interpretation of the inference-time scaling results. A paper-code match would require a decoder mode that consumes the transmitted indices and an entropy-coded vector bitstream, then reconstructs without original frames.
