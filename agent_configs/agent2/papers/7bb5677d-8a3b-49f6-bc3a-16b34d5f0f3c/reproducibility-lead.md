# Reproducibility Lead Report

Paper ID: `7bb5677d-8a3b-49f6-bc3a-16b34d5f0f3c`

Title: `3DGSNav: Enhancing Vision-Language Model Reasoning for Object Navigation via Active 3D Gaussian Splatting`

Role: Reproducibility Lead

Date: 2026-04-25

## Claims Tested

The review team tested the decision-critical claims:

1. 3DGSNav improves aggregate zero-shot object navigation SR/SPL on HM3Dv1, HM3Dv2, and MP3D over state-of-the-art baselines.
2. The Gemini3 planner improves over random frontier selection and Qwen planner variants.
3. The ablations correctly attribute performance to virtual viewpoints, free-viewpoint optimization, annotations, re-verification, planner CoT, and frontier clustering.
4. The real-robot experiments support the stated 69.44% SR and robustness claim.
5. The algorithmic specification is correct enough to reproduce the active 3DGS mapping, viewpoint optimization, and target re-verification pipeline.
6. The novelty framing around 3DGS as VLM memory and free-viewpoint navigation is accurate relative to prior work.

Minimum reproduction target before seeing outcomes:

- Strong: rerun at least one official simulation benchmark or ablation from released code/configs, or independently recover raw SR/SPL from released trajectory logs.
- Partial: recompute table arithmetic and verify at least one implementation path or raw result artifact.
- Weak: table arithmetic only, with no runnable implementation or raw outputs.

Tolerance:

- Table arithmetic: exact up to rounding.
- Empirical reproduction: SR/SPL within reasonable rounding of reported table values under the same episode splits and evaluation settings.
- Method equations: must be dimensionally coherent and consistent with the prose.

## Evidence by Role

### Independent Reproducer A

Reproducer A worked from the official paper artifacts only. Full execution was blocked because the artifact package contains LaTeX source, figures, tables, `paper.pdf`, `source.tar.gz`, and a project page, but no 3DGSNav code, Habitat configs, prompts, seeds, checkpoints, raw trajectories, or logs. The role reproduced table arithmetic for the main benchmark averages, Qwen/random comparisons, ZSON/BeliefMapNav comparisons, and the 25/36 = 69.44% real-world SR.

Outcome: weak reproducibility. Central empirical execution was blocked; arithmetic was reproducible.

### Independent Reproducer B

Reproducer B independently checked table arithmetic, alternative averaging conventions, per-dataset best-baseline comparisons, ablation wording, and episode-count consistency. This role agreed that the headline 13.5% SR and 32.08% SPL gains are recoverable only as unweighted dataset averages against ApexNav's complete three-dataset row. The stronger per-dataset SOTA reading fails on HM3Dv2 SR, where ApexNav reports 76.2 and 3DGSNav reports 75.0. Several percentage values are not cleanly compatible with the stated full episode counts, suggesting undisclosed rounding, subsets, or truncation.

Outcome: weak reproducibility. Independent route confirmed arithmetic consistency under specific conventions and identified interpretation fragility.

### Implementation Auditor

The implementation audit found no executable method release. Koala metadata has `github_repo_url: null` and `github_urls: []`. The local and live project page show `Code(Coming soon)` with an empty GitHub href. The inferred GitHub Pages repository contains only website assets. No code exists for online 3DGS mapping, active perception, frontier extraction, watershed clustering, Dijkstra/FMM planning, viewpoint optimization, prompt generation, YOLOE/GLM re-verification, Habitat evaluation, runtime measurement, or real-robot deployment.

Outcome: decisive artifact blocker. None of the central implementation-dependent claims are independently auditable.

### Correctness Specialist

The correctness pass found no fatal table arithmetic error, but did find major method-specification problems:

- The view alignment loss uses `1 - cos^2(theta)`, which is minimized both when the camera faces the frontier and when it points exactly away.
- The appendix says low-opacity top-down regions are obstacles, apparently inverted relative to 3DGS opacity semantics and inconsistent with low-opacity active-perception regions.
- The panoramic camera intrinsics use image width and horizontal FoV for both axes despite a non-square image and different vertical FoV.
- The guidance trajectory reuses `d_s` for both success radius and obstacle safety threshold and defines a thresholded penalty without a piecewise threshold.
- Several ablation claims use percentage points or full-normalized drops while describing them as percentage improvements.

Outcome: major negative. If the implementation follows the paper literally, central active-perception/viewpoint components are technically flawed; if implementation differs, the paper is not reproducible from the method text.

### Literature Specialist

The literature pass found the integrated ZSON system plausible but the novelty framing overstated. Directly relevant 3DGS navigation/memory/viewpoint prior work includes GaussNav, LagMemo, SplatSearch, and Hierarchical Scoring with 3DGS. GaussNav is already in the bibliography and even appears in commented related-work text, but is not discussed in the final active related-work paragraphs. The claim of being first to systematically incorporate 3DGS representations into VLMs for long-horizon spatial reasoning is too broad. The paper also says it avoids scene abstraction, but still relies on an opacity-derived exploration/occupancy map, clustered frontiers, BEV annotations, Dijkstra trajectories, and FMM.

Outcome: moderate novelty/framing weakness. The contribution is a specific integrated 3DGS/VLM ObjectNav system, not a first use of 3DGS memory/rendered viewpoints for navigation.

## Reproduction Outcome

At least two independent roles reproduced only table arithmetic, not the central empirical result. Neither independent reproducer could rerun a Habitat episode, recover raw SR/SPL from logs, inspect official code, verify prompts, or audit real-robot trials.

Central claim status:

- Benchmark SR/SPL: partially reproduced as table arithmetic; not empirically reproduced.
- VLM comparison: partially reproduced as table arithmetic; not empirically reproduced.
- Ablations: partially reproduced, with denominator/wording errors; not empirically reproduced.
- Real-world SR: arithmetic reproduced as 25/36; real-world execution not reproduced.
- Algorithmic correctness: materially questionable in several equations/definitions.
- Novelty framing: overstated relative to 3DGS navigation prior work.

Overall reproducibility classification: weak.

## Decision Impact

This paper should be materially marked down under a reproducibility-first standard. The table values are not internally random-looking; many arithmetic claims are recoverable from the LaTeX tables. But the central acceptance case is an integrated empirical robotics/navigation system, and that system is not reproducible from the released artifacts. The method text also contains enough correctness errors that the absence of code is more damaging: there is no way to tell whether the implementation silently fixes the paper or inherits the errors.

Recommended verdict range if no new artifacts appear: 3.0 to 4.5.

## Remaining Uncertainty

It is possible that the authors have a working private implementation and that the reported tables reflect real experiments. The evidence package does not permit independent verification. This uncertainty matters because the claimed gains depend on many hidden choices: VLM/API settings, prompts, simulator splits, 3DGS optimizer behavior, detector thresholds, map resolution, navigation planner details, real-robot conditions, and metric computation.

The uncertainty does not rescue the acceptance case. In its current form, the paper asks reviewers to trust a complex closed artifact while the written method has correctness problems and the novelty framing omits close prior work.
