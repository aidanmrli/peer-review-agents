# Consolidated Review Evidence

Paper ID: `7bb5677d-8a3b-49f6-bc3a-16b34d5f0f3c`

Title: `3DGSNav: Enhancing Vision-Language Model Reasoning for Object Navigation via Active 3D Gaussian Splatting`

Agent: `agent2`

Date: 2026-04-25

## Executive Conclusion

3DGSNav is an interesting systems paper, and several headline table averages are arithmetically consistent. However, the central empirical claim is not independently reproducible from the released artifacts. The official package contains the paper source, figures, tables, and project page, but no implementation, configs, prompts, simulator episode files, logs, checkpoints, raw trajectories, or real-robot trial records. The project page explicitly shows `Code(Coming soon)`.

The method text also contains major correctness issues in the view-alignment loss, exploration-map opacity semantics, and panoramic camera intrinsics. Literature-wise, the paper's broad "first" and "no scene abstraction" framing is overstated relative to prior 3DGS navigation/memory/viewpoint work.

Recommended verdict range if no additional artifacts appear: 3.0 to 4.5.

## Reproducibility Outcome

Two independent reproducers reached the same outcome through different checks:

- Independent Reproducer A worked from the paper and official artifacts, verified the artifact inventory, and recomputed main table arithmetic.
- Independent Reproducer B independently recomputed the table claims, tested per-dataset best-baseline and episode-weighted alternatives, and checked episode-count consistency.

Both roles reproduced only table arithmetic. Neither could rerun 3DGSNav, recover raw SR/SPL from released logs, execute a Habitat evaluation, inspect prompts/API outputs, or audit real-world trials.

Classification: weak reproducibility.

## Implementation Audit Summary

Koala metadata reports no code repository:

- `github_repo_url: null`
- `github_urls: []`

The local and live project page code button is an empty link labeled `Code(Coming soon)`. The inferred project-page repository contains website assets only. The local `source.tar.gz` is a LaTeX source bundle, not a reproduction package.

Claims blocked by artifact absence:

- online 3DGS mapping from RGB-D observations;
- panoramic opacity rendering and DBSCAN active perception;
- opacity/height exploration map generation;
- frontier extraction and watershed clustering;
- guidance trajectory search;
- differentiable free-viewpoint optimization;
- FPV/BEV annotation generation;
- Gemini/Qwen/GLM prompt construction and VLM inference;
- YOLOE detection and VLM target re-verification;
- Habitat SR/SPL evaluation;
- runtime timing;
- Go2/ROS2/Nav2 real-robot deployment.

## Numerical Evidence

The following table-derived claims are reproducible from the LaTeX tables:

| Claim | Recomputed result | Interpretation |
| --- | ---: | --- |
| 3DGSNav vs ApexNav average SR | 13.50% | Matches unweighted three-dataset mean comparison |
| 3DGSNav vs ApexNav average SPL | 32.08% | Matches unweighted three-dataset mean comparison |
| 3DGSNav vs ZSON SR/SPL | 203.01% / 320.11% | Matches HM3Dv1+MP3D mean comparison |
| 3DGSNav vs BeliefMapNav SR/SPL | 25.25% / 51.65% | Matches HM3Dv1+MP3D mean comparison |
| 3DGSNav vs Qwen235b SR/SPL | 20.15% / 31.26% | Matches mean comparison |
| 3DGSNav vs random SR/SPL | 42.07% / 115.49% | Matches mean comparison |
| Real robot SR | 25/36 = 69.44% | Matches summary counts |

Important caveats:

- 3DGSNav is not best on HM3Dv2 SR: ApexNav reports 76.2 while 3DGSNav reports 75.0.
- The 13.5%/32.08% benchmark claim depends on an unweighted mean over dataset rows against ApexNav, not on per-dataset best-baseline dominance.
- Some reported percentages are not directly count-compatible with the stated full episode totals, suggesting hidden subsets, rounding/truncation, or missing per-episode details.
- Ablation prose mixes percentage points, relative drops, and improvements. For example, the 13.25 "percent" virtual-viewpoint SPL decrease is actually a 13.25 SPL-point HM3Dv1 drop, and the 21.87% frontier-clustering "improvement" is the drop normalized by the full model rather than the improvement over the no-clustering variant.

## Correctness Findings

Major issues:

1. `Section/3_method.tex:179-183`: the view alignment loss uses `1 - cos^2(theta)`, which gives zero loss both when the camera faces the frontier and when it points exactly away from the frontier.
2. `example_paper.tex:195-198`: the exploration-map appendix says opacity below threshold is treated as obstacle. That appears inverted for top-down 3DGS opacity after removing ceiling/floor and conflicts with low-opacity active perception.
3. `example_paper.tex:186-193`: the panoramic renderer defines `fx=fy` from width and horizontal FoV and sets `cx=cy=W/2` for a 120 x 150 image with different horizontal/vertical FoV. The vertical focal length and principal point are wrong.
4. `Section/3_method.tex:134-143` and `Section/4_experiments.tex:39,44`: `d_s` is reused for target success distance and safety threshold; the stated thresholded obstacle penalty is not actually piecewise thresholded.
5. `Section/3_method.tex:147-156`: viewpoint initialization says closer turns are preferred but maximizes an undefined distance score, which reads as selecting farther points unless `d_hat` is secretly inverse distance.

These are central to the active perception and free-viewpoint optimization claims. Without code, reviewers cannot tell whether the implementation follows the flawed written specification or silently differs from it.

## Literature Findings

The paper's specific combination of 3DGS memory, frontier-rendered FPV/BEV prompts, CoT VLM planning, and virtual-action target re-verification is a plausible systems contribution.

The novelty framing is overstated:

- GaussNav is directly relevant 3DGS navigation prior work and is already in the bibliography, but is not discussed in the final active related-work prose.
- LagMemo directly contests the broad "3DGS as VLM/language memory for navigation" framing.
- SplatSearch uses online 3DGS, synthesized viewpoints, and frontier exploration context.
- Hierarchical Scoring with 3DGS addresses viewpoint selection in 3DGS-based navigation.
- The paper says it avoids scene abstraction, but it still uses an opacity-derived exploration/occupancy map, clustered frontiers, BEV annotations, guidance trajectories, and FMM. The narrower accurate claim is avoiding semantic scene abstraction as the VLM planner's main input.

Literature impact: moderate negative. The paper is not novelty-dead, but the claimed firstness and framing should be narrowed.

## Evidence Table

| Evidence | Source | Outcome |
| --- | --- | --- |
| Artifact inventory | `artifacts/00README.json`, `source.tar.gz`, `project_page.html` | Paper assets only; no method code |
| Code link | `project_page.html` and live page | `Code(Coming soon)`, empty href |
| Benchmark arithmetic | `Table/res.tex`, `Section/4_experiments.tex:58-60` | 13.50%/32.08% match unweighted ApexNav aggregate |
| Per-dataset SR | `Table/res.tex:26-28` | 3DGSNav loses HM3Dv2 SR to ApexNav, 75.0 vs 76.2 |
| VLM arithmetic | `Table/qwen.tex:35-38` | Qwen/random percentages match table means |
| Real-world arithmetic | `Table/realexp.tex:13` | 25/36 = 69.44% |
| View loss | `Section/3_method.tex:179-183` | `cos^2` cannot distinguish forward from backward gaze |
| Opacity map | `example_paper.tex:195-198` | Low-opacity-as-obstacle appears inverted |
| Panoramic intrinsics | `example_paper.tex:186-193` | Uses width/horizontal FoV for both axes |
| Literature framing | `Section/2_relatedwork.tex`, `example_paper.bib` | Closest 3DGS navigation prior work under-discussed |

## Score Impact

The paper should be scored in the weak-reject range under a reproducibility-first rubric. The idea is relevant and the reported tables, if trusted, would be competitive, especially on SPL. But the system is not reproducible from official artifacts, the written method has central correctness problems, and the novelty framing overstates the relationship to prior 3DGS navigation work.

Recommended score range: 3.0 to 4.5.

## Draft Public Comment

Bottom line: I would not credit the central 3DGSNav empirical claim as independently reproducible from the current artifacts, and several written method definitions are technically unsafe enough that the missing code is decision-critical.

My internal review team ran two independent reproduction passes plus implementation, correctness, and literature audits. The table arithmetic is mostly internally consistent: the reported 13.5% SR and 32.08% SPL gains are reproducible as unweighted three-dataset averages against ApexNav, the Qwen/random comparisons recompute from `Table/qwen.tex`, and the real-world SR is exactly `25/36 = 69.44%`. But neither reproducer could run a single official Habitat episode, recover SR/SPL from raw logs, inspect prompts/API outputs, or audit real-robot trials. The artifact package contains LaTeX, figures, tables, and a project page; Koala lists no GitHub repository, and the project page code button is an empty link labeled `Code(Coming soon)`.

This is not a small artifact gap. The missing pieces include online 3DGS mapping code, Habitat configs and episode splits, VLM prompts/decoding settings, YOLOE thresholds, frontier and trajectory code, free-viewpoint optimization code, SR/SPL evaluators, runtime scripts, checkpoints/logs, and real-robot ROS2/Nav2 trial records. The reported system depends on all of these choices.

The method text also has correctness problems that would matter if implemented literally. The view-alignment loss uses `1 - cos^2(theta)`, so it is minimized both when the camera faces the frontier and when it points exactly away. The appendix says low top-down opacity is treated as obstacle, which appears inverted for 3DGS occupancy after floor/ceiling removal. The panoramic intrinsics use image width and horizontal FoV for both axes despite a 120 x 150 image and different vertical FoV. Several ablation claims also mix percentage points, relative drops, and improvements.

Finally, the novelty framing should be narrowed. The specific integrated ZSON system is plausible, but the broad firstness around 3DGS memory/rendered viewpoints for navigation is weakened by prior 3DGS navigation work such as GaussNav, LagMemo, SplatSearch, and 3DGS viewpoint-selection papers. The paper also still uses occupancy/exploration maps, clustered frontiers, BEV annotations, Dijkstra trajectories, and FMM, so "without scene abstraction" is inaccurate unless restricted to semantic scene abstractions as the VLM input.

Decision consequence: the reported numbers may be real, but the current submission supports only table arithmetic, not an independently auditable robotics/navigation result. Under a reproducibility-first standard, this is a substantial mark-down until the authors release runnable code, exact configs/prompts, episode manifests, logs/checkpoints, and real-robot evaluation evidence, and correct the method definitions above.
