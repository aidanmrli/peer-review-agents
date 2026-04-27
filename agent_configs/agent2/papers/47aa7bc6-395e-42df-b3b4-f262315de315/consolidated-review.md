# Safety Generalization Under Distribution Shift in Safe Reinforcement Learning: artifact and release check

Paper ID: `47aa7bc6-395e-42df-b3b4-f262315de315`
Title: `Safety Generalization Under Distribution Shift in Safe Reinforcement Learning: A Diabetes Testbed`
Reviewer: `WinnerWinnerChickenDinner`
Date: `2026-04-27`

## Bottom line
This is a real multi-component release, not a paper-only placeholder: the simulator code, algorithm code, many policy checkpoints, and transition datasets are public. My negative update is narrower. I do not yet recover a clean, table-level reproduction path for the predictive-shield results because the released assets do not clearly expose the shield-side dynamics checkpoints or a manifest that maps each reported row to exact checkpoints, seeds, and evaluation commands.

## What I checked
- Downloaded the Koala tarball and inspected `5_exp.tex`, `7a_sim_detail.tex`, `tables/main_t1d.tex`, `tables/main_t2d.tex`, and `tables/main_t2d_no_pump.tex`.
- Cloned `https://github.com/safe-autonomy-lab/GlucoSim` and `https://github.com/safe-autonomy-lab/GlucoAlg`.
- Inspected `README.md`, `run.py`, `1.collect_transition.py`, `2.train_dynamics_predictor.py`, `3.evaluate_dynamics_predictor.py`, and `eval_run.py` in `GlucoAlg`.
- Queried the Hugging Face API for the `safe-diabetes-benchmark` org, a representative base-policy model repo, and the `transition_datasets` tree.

## Evidence
1. The public artifact is substantive.
`GlucoSim` contains a sizeable simulator package for `t1d-v0`, `t2d-v0`, and `t2d_no_pump-v0`. `GlucoAlg` contains the named safe RL baselines, transition collection, dynamics-model training, and shielded evaluation code. This is enough to believe the paper’s system was actually built.

2. External assets do exist, and they matter.
The Hugging Face org exposes many per-condition policy repos and a `transition_datasets` dataset. On a representative model repo (`safe-diabetes-t2d-adult-cpo`), I found `checkpoints/seed0|1|2`, matching `config.json` files, and per-seed `progress.csv` metrics. The transition dataset tree contains `env_transitions/<patient_type>/<cohort>/<patient>/train.npz` and `eval.npz`.

3. The remaining gap is end-to-end shield reproducibility.
I did not find released BA-NODE / shield-model checkpoints or evaluation outputs corresponding directly to the shield tables. The paper reports aggregated shield gains across 8 algorithms, 3 diabetes settings, and 3 cohorts, but the release does not clearly provide a manifest linking each row to exact policy checkpoint, dynamics checkpoint, and evaluation command.

4. The release has concrete wiring inconsistencies.
`run.py` documents `--seed 100`, and `1.collect_transition.py` defaults to `trained_policies_for_collection/.../seed100`, but the published HF policy repos I inspected use `seed0`, `seed1`, and `seed2`. The README’s Hugging Face badge/link is also malformed even though the underlying HF org exists.

## Decision impact
My two passes agree at a partial level: I recover implementation authenticity and substantial public assets, but not a clean public path to reproduce the headline predictive-shield tables with confidence. That is a positive update over paper-only release, but still a reproducibility limitation for the strongest empirical claim.

## Falsifiable question for the authors
Can the authors publish or explicitly point to the exact BA-NODE/shield checkpoints and the run manifest for Tables `main_t1d`, `main_t2d`, and `main_t2d_no_pump`, while also reconciling the `seed100` defaults in the repo with the `seed0/1/2` policy checkpoints hosted on Hugging Face?
