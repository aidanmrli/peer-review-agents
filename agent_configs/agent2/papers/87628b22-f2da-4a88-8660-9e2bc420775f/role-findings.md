Paper: SoMA: A Real-to-Sim Neural Simulator for Robotic Soft-body Manipulation
Paper ID: `87628b22-f2da-4a88-8660-9e2bc420775f`
Date: 2026-04-26

## Reproducibility lead: central claim and reproduction target

Central claim checked: SoMA is a robot-conditioned Gaussian-splat neural simulator that improves real-to-sim resimulation and generalization on real-world soft-body manipulation by about 20%, with stable long-horizon behavior on rope, cloth, doll, and T-shirt tasks. Reproduction target was the main quantitative claim in `sections/0_abstract.tex:9`, `sections/1_intro.tex:30`, `sections/5_experiments.tex:47-71`, and tables `main_table.tex`, `demo_table.tex`, `ablation_table.tex`.

Outcome: only paper-level arithmetic is reproducible. No released artifact supports rerunning training, reconstruction, or evaluation.

## Reproducer A: artifact-first check

Artifact bundle contains paper sources only: LaTeX, figures, tables, bibliography, and PDFs. No Python code, configs, checkpoints, dataset manifests, metric scripts, or environment files were present after listing and extracting `artifacts/source.tar.gz`.

The project page linked in the abstract points to `https://github.com/Wrioste/SoMA`. That repo currently exposes only a 186-byte `README.md` saying `Coming Soon...`; no code or data is released. This means there is still no runnable simulator despite the public repo link.

Consequence: I could not run reconstruction, train the simulator, reproduce the reported 12 FPS inference, or verify resimulation/generalization metrics from raw trajectories.

## Reproducer B: clean-room/specification check

The method text is not complete enough for faithful clean-room reimplementation. Missing or underspecified items include:

- optimizer, learning-rate schedule, batch sizing, training epochs/steps, and seed control
- concrete value of image-loss mixing weight `lambda` in `sections/4_methods.tex:149-157`
- threshold `tau` for support-force activation in `sections/4_methods.tex:91-95`
- definition/size of `attr_dim` referenced in `sections/3_preliminary.tex:8`
- exact graph construction, edge neighborhoods, and control-point placement beyond the statement that the number of control points is fixed to 30 in `appendix/1_implementation.tex:55`
- dataset sequence identities, train/test split manifests, segmentation prompts, Pi3 reconstruction settings, and object-mask generation details needed to recreate the inputs

Result: a clean-room build would require new implementation choices, not a reproduction of the submitted system.

## Implementation auditor: code/artifact/repo match

The released materials do not match the complexity of the claimed system. The paper depends on ARX-Lift data collection with three D405 cameras, GroundingDINO plus Grounded-SAM2 masking, Pi3 reconstruction, hierarchical graph simulation, and adapted PhysTwin/GausSim baselines (`appendix/1_implementation.tex:5-22`, `46-99`). None of the corresponding scripts or configs are released.

The main and appendix tables are present, but there are several presentation inconsistencies that matter for trust:

- `ablation_table.tex` caption says `w/o MRT`, while the column label is `w/o MRF`
- `ablation_table.tex` misspells `General` as `Genral`
- `hierarchy.tex` has the wrong caption: it says `Quantitative results on the cloth folding task` although it actually lists clustering parameters

These are not fatal alone, but they reinforce that the release is still manuscript-only.

## Correctness specialist: methods, metrics, and conclusion risks

The headline `20% improvement` is not directly recoverable from the released main table. Using the best baseline per reported metric from `main_table.tex`, SoMA's relative gains are:

- resimulation: 12.75%, 17.33%, 5.74%, 2.53%, 36.05%
- generalization: 12.50%, 18.45%, 5.11%, 2.76%, 32.61%

The mean over these ten relative gains is about 14.58%, not 20%. Some individual metrics exceed 20%, but the paper does not specify which aggregation defines the abstract/introduction claim.

Additional decision-relevant risks:

- evaluation is entirely observation-based; there is no full 3D ground truth, only depth proxies on valid object regions (`sections/5_experiments.tex:28-33`, `appendix/1_implementation.tex:105-113`)
- T-shirt evaluation omits GausSim entirely in `demo_table.tex`, leaving the harder task compared only against PhysTwin

## Literature specialist: novelty/framing against prior work

The paper's positioning is plausible: robot-conditioned real-to-sim soft-body simulation with Gaussian splats is a meaningful systems direction distinct from passive 4D reconstruction and from state-only Gaussian-splat simulators. The novelty concern is not obvious firstness but support: the submission asks readers to trust a fairly elaborate pipeline without releasing the implementation needed to judge whether the proposed interaction-aware ingredients, rather than unreleased data and engineering choices, drive the gains.

## Checks actually run

- `tar -tzf papers/87628b22.../artifacts/source.tar.gz`
- extracted source tree and listed files under `artifacts/source`
- read `sections/4_methods.tex`, `sections/5_experiments.tex`, `appendix/0_discussion.tex`, `appendix/1_implementation.tex`
- read `tables/main_table.tex`, `tables/demo_table.tex`, `tables/ablation_table.tex`, `tables/ablation_k.tex`, `tables/hierarchy.tex`
- fetched `https://city-super.github.io/SoMA/`
- queried `https://api.github.com/repos/Wrioste/SoMA/contents`
- fetched `https://raw.githubusercontent.com/Wrioste/SoMA/main/README.md`
