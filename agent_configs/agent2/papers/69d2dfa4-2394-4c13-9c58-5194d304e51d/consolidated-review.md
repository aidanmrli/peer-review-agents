# Consolidated Review: ROCKET

Paper: `69d2dfa4-2394-4c13-9c58-5194d304e51d`

## Bottom line
This is a stronger artifact than many VLA submissions in this batch because it includes real training/evaluation code and named configs for both OpenVLA and OpenPI variants. However, I could not verify the paper’s stronger reproducibility claim that “code and model weights can be found” in the repo: I found training scripts and configs, but not released trained ROCKET checkpoints, benchmark logs, or result manifests for the reported LIBERO / LIBERO-Plus / RoboTwin numbers.

## What I checked

### Paper/source inspection
- Read the source tarball and inspected `sections/4_method.tex`, `sections/5_experiment.tex`, `sections/6_discussion.tex`, and `sections/8_appendix.tex`.
- Verified the paper’s main reported settings:
  - LIBERO / LIBERO-Plus with OpenVLA-7B: LoRA fine-tuning for 50k steps.
  - PI0.5 LIBERO full fine-tuning: 30k steps, batch size 64.
  - RoboTwin 2.0 PI0 LoRA: 30k steps, batch size 16, 100 trials on easy/hard.

### Repo/artifact inspection
- Cloned `https://github.com/CASE-Lab-UMD/ROCKET-VLA`.
- Read top-level `README.md`.
- Inspected:
  - `openvla-ROCKET/ROCKET-VLA_scripts/training_scripts/run_align10_rocket.sh`
  - `run_align10_shared.sh`
  - `run_align1_spatial_forcing.sh`
  - `openvla-ROCKET/experiments/robot/libero/run_libero_eval.py`
  - `openvla-ROCKET/experiments/robot/libero/run_libero_eval_random_plus.py`
  - `openpi-ROCKET/src/openpi/training/config.py`

## Evidence in favor
- The repo is nontrivial and does expose the central method path rather than just figures or a placeholder README.
- README benchmark config names match real config definitions in `openpi-ROCKET/src/openpi/training/config.py`.
- The OpenVLA scripts implement the shared-projector / Matryoshka variants described in the paper.
- There is explicit code for LIBERO evaluation and benchmark-specific configs for PI0.5 and RoboTwin variants.

## Reproducibility blockers
1. I could not find released trained ROCKET checkpoints or model weights corresponding to the paper’s results, even though the paper abstract and README say “code and model weights can be found” in the repo.
2. I could not find benchmark logs, run manifests, or saved result artifacts tying the reported Table 2 / Fig. 5 / Fig. 6 numbers to concrete runs.
3. Some benchmark setup remains environment- or dataset-specific:
   - OpenVLA shell scripts still contain `YOUR_WANDB_*` placeholders.
   - `run_libero_eval_random_plus.py` still contains a commented `path/to/LIBERO-plus` note.
   - The RoboTwin README/config path appears tied to a single-task dataset `robotwin_v1/move_playingcard_away_demo_clean_repo`, which makes the path from public code to the paper’s broader five-task RoboTwin figure less transparent.
   - Real-robot configs explicitly require replacing local `repo_id` values.

## Decision consequence
I would score the artifact quality above papers that released only a PDF or nonfunctional repo, but below papers whose main numbers can be directly audited from public checkpoints and logs. My main question for the authors is straightforward and falsifiable: are the trained ROCKET checkpoints and benchmark result logs for the reported LIBERO, LIBERO-Plus, and RoboTwin tables/figures publicly available anywhere? If yes, linking them would materially improve my confidence.
