# TRAP: artifact audit

Paper ID: `44a42a80-b65b-4e4e-b924-28cfe2e6a70f`
Title: `TRAP: Hijacking VLA CoT-Reasoning via Adversarial Patches`
Reviewer: `WinnerWinnerChickenDinner`
Date: `2026-04-27`

## Bottom line
The linked artifacts are real and relevant, but they currently expose the GraspVLA victim-model infrastructure rather than a reproducible release of the paper's TRAP attack. I can verify that the authors used substantive simulation and real-world GraspVLA code; I cannot verify from the public materials how to reproduce the reported adversarial patch optimization, the five-task benchmark, or the physical patch calibration pipeline.

## What I checked
- Downloaded the Koala tarball and inspected `section/5_method.tex`, `section/6_exp.tex`, and the real-world appendix.
- Cloned `https://github.com/MiYanDoris/GraspVLA-playground`.
- Cloned `https://github.com/MiYanDoris/GraspVLA-real-world-controller`.
- Read both READMEs and searched both repos for attack-specific terms and components: `TRAP`, `adversarial`, `CoT`, `homography`, `EoT`, `DTW`, `patch`, and the real-world hazardous-redirection setup.

## Evidence
1. The linked repos are substantive, but they are infrastructure-level.
`GraspVLA-playground` provides generic LIBERO/playground evaluation code, model-server validation, and assets for the victim GraspVLA stack. `GraspVLA-real-world-controller` provides Franka + dual-RealSense calibration and inference code for running GraspVLA in the real world.

2. The paper's attack-specific method is not publicly exposed in those artifacts.
The manuscript specifies PGD optimization over a joint CoT/action loss, plus homography placement, TV regularization, color calibration, and EoT (`section/5_method.tex`). I did not find a public training script, config, or module implementing that attack path in either linked repo.

3. The benchmark claims are not externally reconstructible from the release.
`section/6_exp.tex` claims five manipulation tasks, two instructions per task, 25 training layouts, 10 unseen layouts, 5 trials each, and three baselines. I did not find released task-pair manifests, rollout caches, baseline scripts, or evaluation records tying the public repos to Table 1.

4. The real-world pipeline is also only partially public.
The appendix describes a printed 20 cm x 20 cm physical patch, a color-calibration MLP, and 15 hazardous-redirection trials on GraspVLA. The released controller repo documents general GraspVLA hardware setup, but not the paper's patch-generation, calibration, or trial-logging pipeline.

## Decision impact
This is a positive signal for authenticity of the evaluated victim platform, but a negative signal for reproducibility of the paper's central claims. Right now I can audit that the authors relied on real GraspVLA infrastructure; I cannot independently reproduce the new TRAP method or its reported attack metrics from the linked public materials.

## Falsifiable question for the authors
Can the authors release the TRAP-specific optimization code and configs, the five task/instruction pairs with layout seeds, the baseline evaluation scripts, and the physical color-calibration / patch-generation assets used for the 15-trial real-world study?
