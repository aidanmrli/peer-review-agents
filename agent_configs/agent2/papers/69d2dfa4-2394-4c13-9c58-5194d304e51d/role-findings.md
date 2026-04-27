# ROCKET Role Findings

## Reproducibility lead: central claim and reproduction target
Central claim checked: ROCKET reaches state-of-the-art LIBERO performance at about 4% of the prior compute budget, and also improves LIBERO-Plus and RoboTwin. Reproduction target was not rerunning training; it was verifying whether the public artifact is sufficient for an independent reviewer to reproduce or at least audit the reported Table 2 / Fig. 5 / Fig. 6 numbers.

## Reproducer A: artifact-first check
The release is substantive. The public repo contains real code for `openvla-ROCKET` and `openpi-ROCKET`, training scripts, profiling scripts, evaluation scripts, and named configs matching the README. Evidence:

- `README.md` describes both OpenVLA and OpenPI workflows and claims code and model weights are available.
- `openvla-ROCKET/ROCKET-VLA_scripts/training_scripts/run_align10_rocket.sh` and related ablation scripts expose the main OpenVLA training commands.
- `openvla-ROCKET/experiments/robot/libero/run_libero_eval.py` and `run_libero_eval_random_plus.py` provide evaluation code for LIBERO and LIBERO-Plus.
- `openpi-ROCKET/src/openpi/training/config.py` defines the README config names for PI0.5 LIBERO and PI0 RoboTwin.

Main artifact limitation: I could not find released ROCKET checkpoints / trained weights / result logs inside the repo despite the paper abstract and README saying “code and model weights can be found” in the repo. A `find` over the clone for checkpoint-like files at shallow depth returned nothing. The README only points users to external base-model weights such as OpenVLA-7B and VGGT, not to trained ROCKET checkpoints corresponding to the reported results.

## Reproducer B: clean-room/specification check
The paper source is fairly specific about method components and benchmark settings. The training durations and task setup are stated in `sections/5_experiment.tex`: 50k-step LoRA tuning for OpenVLA LIBERO/LIBERO-Plus, 30k-step full fine-tuning for PI0.5 LIBERO, and 30k-step LoRA tuning for RoboTwin. The code broadly matches those settings:

- OpenVLA scripts use `max_steps 50005` and 10 alignment pairs.
- OpenPI LIBERO configs use `num_train_steps=30_000`.
- RoboTwin configs use `num_train_steps=30_000`.

However, exact auditability of the headline results is still weak because the release does not include the trained outputs, run manifests, or logs tied to the reported tables/figures. The README’s cost table and benchmark summaries therefore cannot be checked end-to-end from the released artifact alone.

## Implementation auditor: code/artifact/repo match
Positive matches:

- README benchmark config names are real and defined in `openpi-ROCKET/src/openpi/training/config.py`.
- LIBERO / LIBERO-Plus data paths are represented in the codebase, including `libero_plus_mixdata`.
- The core ROCKET mechanisms appear implemented, including shared projector / layer alignment controls.

Audit concerns:

- The OpenVLA training shell scripts still contain placeholders such as `YOUR_WANDB_ENTITY`, `YOUR_WANDB_PROJECT`, and `YOUR_RUN_ID`.
- `openvla-ROCKET/experiments/robot/libero/run_libero_eval_random_plus.py` still contains a commented local-path note `# sys.path.append("path/to/LIBERO-plus")`, suggesting some benchmark setup remains environment-specific.
- The README states the RoboTwin config is a single-task dataset (`move_playingcard_away`) rather than an obvious public package for reproducing the paper’s full five-task benchmark claim.
- Real-robot configs explicitly depend on local datasets and manual `repo_id` replacement, so those claims are not independently reproducible from the public release.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
I did not identify an obvious code-paper contradiction in the main ROCKET mechanism. The main correctness risk is evidentiary rather than algorithmic: the paper’s broad claims about reproducibility and released “model weights” are stronger than what the public artifact currently substantiates. Without trained checkpoints or logs, a reader cannot directly verify that the reported 98.5 LIBERO average, 81.7 LIBERO-Plus average, or RoboTwin figure values actually arise from the named configs.

## Literature specialist: novelty/framing against permitted prior work
The framing against single-layer alignment baselines is coherent in the source and the repo does expose corresponding baselines. My concern is not novelty inflation; it is that the release quality currently supports “method released” more than “results reproducible.” If trained ROCKET checkpoints and benchmark logs exist elsewhere, my assessment would improve materially.
