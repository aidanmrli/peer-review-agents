# Literature Specialist Report

## Claim Tested

Whether the paper's novelty framing is supported relative to permitted prior work cited in the manuscript and available by release.

## Evidence

The manuscript itself cites the relevant prior lines:

- INR compression: COIN and NeRV are discussed in `artifacts/chapters/background.tex:29`.
- NVRC is discussed as explicit quantized latents plus implicit neural decoder in `artifacts/chapters/appendix.tex:153-156`.
- GIVIC is cited as diffusion-process INR compression in `artifacts/chapters/background.tex:29`.
- DreamBooth and LoRA-based personalization as visual memory are discussed in `artifacts/chapters/appendix.tex:160-161`.
- Uni-LoRA is acknowledged as closely related to one-vector LoRA mapping in `artifacts/chapters/method_zongyu.tex:113-115` and `artifacts/chapters/introduction_zongyu.tex:15`.
- Diff-C and relative entropy coding are acknowledged in `artifacts/chapters/method_zongyu.tex:232-236` and `artifacts/chapters/appendix.tex:319-340`.

## Findings

The novelty is best characterized as a systems-level combination: per-signal LoRA/UniLoRA adaptation, entropy-constrained one-vector storage, and diffusion/relative-entropy-style scaling for compression. That combination appears meaningful and is not reducible to any single cited component.

The paper is less convincing if read as a new primitive for "visual representation" or a standalone codec standard. LoRA as memory, one-vector LoRA parameter sharing, INR compression, and diffusion compression are all acknowledged precedents. The empirical contribution therefore depends strongly on the quality and reproducibility of the benchmark curves.

The baseline discussion is incomplete for a compression paper. The main curves compare against VTM/HM, DCVC-RT, and GLC-Video (`artifacts/chapters/experiments.tex:27-33`), while close INR/diffusion compression predecessors such as NVRC and GIVIC are mostly discussed in related work, not directly benchmarked in the main quantitative comparison. This does not invalidate the method, but it weakens the claim of strong compression performance as a field-level result.

## Severity

Medium. Novelty exists as integration, but the paper should not receive full novelty credit unless the empirical comparison and reproducibility are strong enough to distinguish it from the cited predecessor families.
