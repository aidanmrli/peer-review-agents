# Role Findings: 0149e35f-24f1-4fa4-8de0-6fb6d0016389

## Central claim and reproduction target
- Reproduction target: the paper's claim that NPIM is a compact, trainable zeroth-order Ising-machine heuristic with competitive runtime and solution quality, especially the Table 1 / G-set settings that rely on a PyTorch implementation, parallel restarts, and Dynamic Anisotropic Smoothing (DAS) training.

## Paper and artifact evidence checked
- Koala paper metadata: no `github_repo_url`; only PDF and tarball are attached.
- Tarball contents checked with `tar -tzf`: `00README.json`, `main_arxivs.tex`, `appendix.tex`, `param_sweep.tex`, styles, and figures only. No code, configs, logs, or checkpoints.
- Source inspection via `rg` and `sed` on the extracted LaTeX:
  - `main_arxivs.tex` states Table 1 uses PyTorch on an A100 and evaluates `dNPIM (top 30)`.
  - `appendix.tex` Section `Details on Parameter Optimizer (Dynamic Anisotropic Smoothing)` gives the DAS estimators and reports `B=20`, `R=400`.
  - `appendix.tex` benchmark details specify `T_c=20`, `D=3`, `M=3`, `T=N` or `4N` for planar graphs.

## Reproducibility result from the smallest meaningful check actually run
- I reproduced the paper's artifact packaging state only: the public release is manuscript-only.
- I could verify some benchmark hyperparameters from source, but I could not reconstruct the training/inference pipeline because the release does not include executable implementation artifacts.

## Implementation or correctness risks
- The manuscript gives the DAS estimator formulae but does not expose runnable optimizer code, optimizer step-size settings, random seeds, or the initialization of the exploration state `theta_L` / training state used to produce the reported trained policy.
- The Table 1 wall-clock comparison depends on an unreleased PyTorch implementation and a `top 30` restart protocol, so the practical runtime claim is not externally auditable from the released materials.
- The reward definition depends on an evolving best-so-far energy `E_0`, which is described conceptually but not exposed through logs or checkpoints.

## Novelty or framing context
- This is a reproducibility-only check; I did not assess novelty beyond whether the released artifact supports the paper's empirical claims.

## Decision impact
- The paper may still contain a real algorithmic contribution, but the current release is insufficient to independently reproduce the zeroth-order training setup or the runtime comparison as reported. This weakens confidence in the empirical evidence rather than the conceptual idea itself.
