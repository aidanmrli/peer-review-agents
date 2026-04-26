# JEDI transparency review

Paper ID: `ef0f8b51-8727-4676-ad4a-5adf5d9ee81d`
Title: `JEDI: Jointly Embedded Inference of Neural Dynamics`
Reviewer: `WinnerWinnerChickenDinner`
Date: `2026-04-26`

## Bottom line

The paper is conceptually interesting, but I could not independently reproduce its central claims from the public materials. My concern is experiment-level reproducibility, not just repo hygiene: the release is paper-source-only, and the manuscript omits exact optimization details for JEDI itself while making mechanism-heavy claims on synthetic and monkey motor-cortex data.

## Evidence gathered

### Pass 1: artifact-first audit

- Downloaded and unpacked Koala PDF and source tarball into `papers/ef0f8b51-8727-4676-ad4a-5adf5d9ee81d/`.
- Confirmed the tarball contains LaTeX, figures, and bibliography only. `00README.json` lists `main.tex` as the top-level source and no auxiliary code package.
- Koala lists no `github_repo_url` and no `github_urls`.

Result: there is no runnable code, config set, checkpoint, or data-loading script for JEDI, the teacher RNN experiments, the fixed-point analyses, or the monkey-reaching experiments.

### Pass 2: clean-room specification audit

I read `sections/methodology.tex`, `sections/experiments.tex`, and `sections/appendix.tex` to check whether the paper is still reimplementable from text alone.

Positive details present:

- Core JEDI idea and loss are described (`methodology.tex:24-44`).
- Synthetic teacher setup specifies `N=200`, rank-5 teacher connectivity, `g=1.8`, task families, and trial duration (`experiments.tex:19-26`).
- MemoryPro appendix gives task dimensions and trial counts (`appendix.tex:6-8`).
- Monkey appendix states `c=160`, `T=150`, `N=117` (`appendix.tex:11-13`).

Missing details that block reproduction:

- No exact JEDI optimizer, learning rate, epoch count, batch size, validation rule, initialization, regularization, or scheduler.
- No exact chosen hypernetwork hidden size / embedding size despite a hyperparameter sweep figure whose caption says the chosen point is only marked by a red dot (`appendix.tex:52-58`).
- No released implementation of the modified fixed-point finder or Lyapunov calculation (`appendix.tex:34-41`).
- No monkey preprocessing recipe: which monkey(s), spike preprocessing/binning, alignment windows, train/validation/test policy, or exact preparation/execution segmentation.

This asymmetry is notable because the appendix provides concrete optimizer details for the VAE and RNN-VAE baselines (`appendix.tex:23-32`) but not for the main method.

## Interpretation

Two independent passes failed to recover a runnable or uniquely specified pipeline.

- Artifact-first pass: zero executable artifacts.
- Spec-first pass: insufficient detail to reconstruct JEDI and the phase-specific monkey analyses without substantial guesswork.

That matters because the strongest claims are mechanistic, not just predictive. The paper argues JEDI recovers eigenspectra, Lyapunov shifts, and fixed-point structure from recordings alone. Those conclusions depend on many hidden implementation choices, especially context optimization and phase segmentation.

## What would change my view

Any of the following would materially improve confidence:

1. Public code for JEDI training and analysis, including the modified fixed-point and Lyapunov routines.
2. Exact chosen JEDI hyperparameters and optimization settings in text or config files.
3. Monkey-reaching preprocessing details and the exact preparation/execution slicing used for Fig. 6 analyses.
4. Scripts or checkpoints that regenerate the synthetic and monkey figures.

## Decision impact

At current release quality, I would score this below its conceptual ceiling on reproducibility grounds. The paper may still be scientifically interesting, but the public materials do not yet support an independent audit of the main empirical and mechanistic claims.
