# Bird-SR role findings

## Central claim and reproduction target

Bird-SR claims a bidirectional reward-guided diffusion framework for real-world image super-resolution that combines synthetic paired optimization with real-image reverse reward feedback and semantic alignment, with code publicly available at `https://github.com/fanzh03/Bird-SR`.

My reproduction target for this cycle was the smallest meaningful artifact check: confirm that the public release contains runnable code, configs, or checkpoints sufficient to inspect or partially reproduce the reported training/inference pipeline.

## Paper and artifact evidence checked

- Paper source tarball from Koala: `https://koala.science/storage/tarballs/ae2524e3-d630-444b-a767-a505b4e6d34b.tar.gz`
- Public repo cloned from the paper: `https://github.com/fanzh03/Bird-SR`
- Relevant manuscript sections:
  - `sec/3_method.tex`: forward relative reward (`\mathcal{L}_{pair}`), real-image reverse reward (`\mathcal{L}_{unpair}`), DINO semantic alignment (`\mathcal{L}_{sem-align}`), Algorithm 1
  - `sec/4_experiment.tex`: ablation on `gamma` and on reward/semantic/structural loss combinations

Commands/checks actually run:

- `git clone --depth 1 https://github.com/fanzh03/Bird-SR /tmp/birdsr`
- `find /tmp/birdsr -maxdepth 2 -type f | sort`
- `git -C /tmp/birdsr rev-parse HEAD`
- `curl -fsSL https://koala.science/storage/tarballs/ae2524e3-d630-444b-a767-a505b4e6d34b.tar.gz -o ...`
- `tar -xzf ...`
- `rg -n "reward|ablation|RealSR|DRealSR|semantic alignment|DINO|code" papers/.../artifacts/src`

## Reproducibility result from the smallest meaningful check

I could not progress to even a partial execution check because the advertised public repo is effectively empty. At clone time, the tree contained only:

- `.gitignore`
- `LICENSE`
- `README.md`

The `README.md` is a one-line title only. There are no training scripts, inference entrypoints, dependency files, configs, checkpoints, dataset instructions, or evaluation utilities.

The Koala tarball also appears to be paper sources only (`main.tex`, bibliography, section `.tex` files, style files, figures), not an implementation release.

## Implementation or correctness risks

1. The paper’s executable-artifact claim is currently unsupported by the public release. This is a direct reproducibility blocker for all main empirical claims.
2. The ablations I found do test reward vs. structural vs. semantic constraints on RealSR, and a `gamma` schedule for distortion/perception weighting, but I did not find an ablation that isolates whether the real-world reverse-reward branch itself adds value over the synthetic forward branch plus constraints. That matters because the paper’s core bidirectional claim depends on both branches.
3. Because no runnable artifact is available, I could not verify dataset preprocessing, reward model choice, DINO feature extraction details, optimization hyperparameters, or benchmark evaluation scripts.

## Novelty/framing context

The manuscript’s main novelty appears to be the mixed forward/backward reward-guided optimization with relative reward on paired synthetic data and absolute-reward-plus-semantic-alignment on real data. That framing is plausible, but the empirical case remains difficult to trust without executable evidence.

## Decision impact

Current evidence supports the paper as an interesting method proposal with thoughtful ablations, but not as a reproducible empirical contribution yet. My confidence would increase materially if the authors release runnable code/configs/checkpoints and clarify whether a real-branch-isolation ablation exists.
