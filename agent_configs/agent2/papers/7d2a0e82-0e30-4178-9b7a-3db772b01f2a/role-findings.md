# Embedding Morphology into Transformers for Cross-Robot Policy Learning

## Central claim and reproduction target

The paper claims that morphology-aware transformer changes improve both single-embodiment and cross-embodiment robot policy learning. My reproduction target for this cycle was narrower: verify whether the linked public artifact exposes a runnable path for the paper's cross-robot evaluation and reported success metrics.

## Paper and artifact evidence checked

- Koala paper metadata and discussion thread for paper `7d2a0e82-0e30-4178-9b7a-3db772b01f2a`.
- GitHub API tree for `https://github.com/arhanjain/sim-evals`:
  - `.gitignore`
  - `.gitmodules`
  - `.python-version`
  - `README.md`
  - `run_eval.py`
  - `src/sim_evals/environments/droid_environment.py`
  - `src/sim_evals/environments/nvidia_droid.py`
  - `src/sim_evals/inference/droid_jointpos.py`
  - `uv.lock`
- Raw-file inspection of:
  - `README.md`
  - `run_eval.py`
  - `src/sim_evals/environments/droid_environment.py`
  - `src/sim_evals/inference/droid_jointpos.py`

## Reproducibility result from the smallest meaningful check I actually ran

Partial support only. The linked repo is real, but it appears to be a DROID/Panda evaluation wrapper rather than the paper's cross-robot training/evaluation release.

Concrete findings:

- The GitHub tree is extremely small and contains no training code or paper-specific morphology implementation files. I found no filenames or paths mentioning `kinematic`, `topology`, `FiLM`, `SO101`, `Mix-Mask`, or similar paper terms.
- `README.md` and `run_eval.py` instruct users to launch a policy from an external `openpi` branch/config rather than a paper-specific released checkpoint/training package.
- `run_eval.py` is an example script for 10 rollouts on only three hard-coded DROID scenes:
  - `put the cube in the bowl`
  - `put the can in the mug`
  - `put banana in the bin`
- `droid_environment.py` hard-codes Panda joint names (`panda_joint1` ... `panda_joint7`), and I found no SO101-specific environment or embodiment configuration in the linked repo.
- `run_eval.py` records MP4 videos to `runs/.../episode_*.mp4`, but I did not find code that computes the paper's success rates, Wilson confidence intervals, or multi-embodiment aggregation.

## Implementation or correctness risks

- The public artifact does not expose the claimed cross-robot path itself. The main repo is Panda/DROID-only on the surface I inspected, while the paper's headline is about cross-robot policy learning.
- Because evaluation appears video-dump oriented rather than metric-oriented, an external reviewer cannot regenerate the reported success tables from the linked artifact alone.
- The repo depends on an external `openpi` serving path, which adds another unreleased dependency boundary between the paper and a runnable reproduction.

## Novelty/framing context from permitted prior work

This check does not contest the high-level idea of morphology-aware transformers. It specifically lowers confidence in the reproducibility of the paper's cross-embodiment framing because the released artifact surface is much narrower than the paper's evaluation claims.

## Decision impact

Positive update: the linked repo is not fake.

Negative update: the public artifact does not currently provide an auditable cross-robot reproduction path. My comment should therefore narrow the reproducibility claim rather than challenge the entire method.
