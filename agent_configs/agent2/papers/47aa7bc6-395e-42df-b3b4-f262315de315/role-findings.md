# Reproducibility lead: central claim and reproduction target
Central claim checked: test-time predictive shielding restores safety under distribution shift across eight safe RL algorithms, three diabetes settings, and three age cohorts, with BA-NODE as the learned dynamics model that powers the shield. Reproduction target was whether the released artifact exposes an end-to-end path to the reported shield tables rather than only partial components.

## Reproducer A: artifact-first check
Public assets are materially stronger than paper-only release:
- `GlucoSim` contains a substantial JAX simulator package with T1D, T2D, and T2D-without-pump environments plus parameter CSVs.
- `GlucoAlg` contains training (`run.py`), transition collection (`1.collect_transition.py`), dynamics training (`2.train_dynamics_predictor.py`), and evaluation (`eval_run.py`) code.
- The Hugging Face org `safe-diabetes-benchmark` exposes many base-policy model repos and a `transition_datasets` dataset. Via the HF API I confirmed per-condition model repos with checkpoints/configs/metrics and a dataset tree containing `env_transitions/<patient_type>/<cohort>/<patient>/train|eval.npz`.

## Reproducer B: clean-room/specification check
I do not yet recover a clean table-reproduction path for the shielding results:
- The paper reports shield gains aggregated over 8 algorithms, 3 diabetes settings, and 3 cohorts. The public GitHub repos do not include shield evaluation outputs, released BA-NODE checkpoints, or a manifest tying each reported table row to exact checkpoint epochs and evaluation commands.
- HF model repos appear to host base-policy checkpoints (`seed0/1/2`), but I found no separate HF model repos for BA-NODE or shield checkpoints.
- The README’s “Models and Datasets” badge/link is malformed even though the underlying HF org exists, which weakens discoverability for a reader following the repo literally.

## Implementation auditor: code/artifact/repo match
The implementation is substantive and aligned with the paper:
- `run.py` exposes the named safe RL baselines.
- `eval_run.py` implements shielded evaluation and computes TIR, risk index, and CV.
- `1.collect_transition.py`, `2.train_dynamics_predictor.py`, and `3.evaluate_dynamics_predictor.py` define the intended pipeline for training the shield-side predictor.

Concrete release mismatches remain:
- `run.py` examples use `--seed 100`, and `1.collect_transition.py` defaults to `trained_policies_for_collection/.../seed100`, but the HF policy repos I inspected expose `seed0`, `seed1`, and `seed2`.
- `GlucoAlg` ships no in-tree `saved_models`, `saved_files/env_transitions`, or `saved_files/dynamics_predictor`; reproduction depends on locating and wiring external assets correctly.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
The main risk is reproducibility completeness, not apparent vaporware. The paper’s strong multi-setting table claims are plausible given the released components, but I cannot audit whether the exact BA-NODE model and shield evaluation runs behind Tables `main_t1d`, `main_t2d`, and `main_t2d_no_pump` are publicly recoverable from the current artifact set.

## Literature specialist: novelty/framing against permitted prior work
No literature-specific blocker emerged from the artifact pass. The decision-relevant issue is that the public release supports method authenticity and partial rerunnability, but not yet a straightforward reproduction of the headline shield tables.
