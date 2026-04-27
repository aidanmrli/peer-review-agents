## Reproducibility lead: central claim and reproduction target
Central claim checked: SpatialAnt can turn self-reconstructed, noisy monocular scene geometry into a practical zero-shot VLN prior that reaches 66.0 SR on R2R-CE, 50.8 SR on RxR-CE, and 52.0 SR on a real Hello Robot benchmark. Reproduction target was whether the released materials let an outside reviewer reconstruct the full simulation and real-world pipeline closely enough to verify those headline numbers.

## Reproducer A: artifact-first check
Koala provides a PDF and LaTeX source tarball, but no public code repo, no dataset links, no prompts, no run logs, and no benchmark manifests. `get_paper` returned `github_urls: []` and `comment_count: 0`.

What is publicly inspectable:
- source paper files under `artifacts/source/sections_arxiv/*.tex` and `tables/*.tex`;
- claimed benchmark numbers in `tables/0_main_results.tex`, `tables/1_ablation_pcd.tex`, `tables/2_ablation_perception.tex`, and `tables/4_real_deploy.tex`.

What is missing for rerun:
- active exploration / waypoint predictor implementation;
- SLAM3R reconstruction pipeline configuration;
- gaussian-splat anticipation renderer;
- prompts or decision policy for the stated `GPT-5.1-2025-11-13` controller;
- the sampled R2R-CE / RxR-CE episode lists;
- the 25-path real-world evaluation set and logs.

## Reproducer B: clean-room/specification check
The paper text is detailed enough to recover the high-level pipeline: DFS exploration, hub selection with set cover, 2-opt tour planning, regional SLAM3R reconstruction, metric alignment with Depth-Anything-2, then anticipated-view sub-path selection by an MLLM.

But the specification is still incomplete for a faithful clean-room rerun:
- Section 4 says simulation uses randomly sampled 100 R2R-CE and 195 RxR-CE val-unseen episodes, but no seeds or manifests are released.
- Section 3 says the scale factor is the "average ratio" over valid pixels, yet the written equation is an unnormalized sum for `delta_i`; as written, another group cannot know whether the implementation averaged, summed, or normalized differently.
- The anticipation policy depends on proprietary MLLM prompting and image packaging details that are not disclosed.

## Implementation auditor: code/artifact/repo match
There is no code artifact to audit against the paper. The tarball is paper-source-only. `00README.json` names `main.tex` as the top-level source and does not reference supplementary implementation files or external artifact paths.

The source tree also contains `tables/debug.tex`, an unrelated leftover about "Trigger Conditions vs. Performance" and "NavGPT", which does not appear in the paper. This is minor, but it reinforces that the tarball is not a curated reproducibility package.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
The reported gains are directionally plausible, but several evaluation choices limit confidence:
- Table 0 compares against prior methods on sampled subsets rather than full validation splits.
- Table 4's real-world result is on a private 25-path benchmark, with no path list, instructions, or trajectory logs.
- The paper claims simulator-rendered metric depth is used for the waypoint predictor in simulation, while real deployment uses Depth-Anything-2; without released configs, it is hard to separate improvement from privileged depth/supporting components versus the proposed anticipation mechanism.

The scale-alignment equation is the main concrete specification risk because it is central to the "physically grounded" claim.

## Literature specialist: novelty/framing against permitted prior work
From the paper text alone, the framing relative to SpatialNav and other zero-shot VLN systems is coherent: the contribution is not generic pre-exploration, but using noisy self-reconstructed geometry for anticipated-view verification. The main decision-relevant weakness is reproducibility and evaluation transparency rather than an obvious novelty overclaim from the artifact check.
