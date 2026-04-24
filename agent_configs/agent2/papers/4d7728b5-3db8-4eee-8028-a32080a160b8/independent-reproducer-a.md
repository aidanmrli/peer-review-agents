# Independent Reproducer A Report

## Paper ID and Title

- Paper ID: `4d7728b5-3db8-4eee-8028-a32080a160b8`
- Title: "Scalable Simulation-Based Model Inference with Test-Time Complexity Control"

## Assigned Role

Independent Reproducer A. I performed an independent, paper-and-official-artifact-only pass starting from the LaTeX source, PDF artifact, and linked repository clone. I did not inspect Independent Reproducer B's reasoning and did not use OpenReview, citation counts, social media, acceptance signals, or future/leakage sources.

## Task Scope

I attempted to verify the central reproducibility claim: PRISM performs scalable simulation-based joint inference over model structure and parameters, supports test-time complexity control through a tunable model prior, and obtains the reported symbolic-regression and dMRI empirical metrics.

Because the official repository clone is non-executable, I reduced the reproduction target to the smallest meaningful checks available from the provided artifacts:

1. Check whether the official repo contains runnable commands, configs, checkpoints, data, or scripts sufficient to reproduce any reported metric.
2. Recompute the paper's stated model-space and training-simulation arithmetic for the symbolic-regression scaling claim.
3. Verify the stated symbolic prior's monotone control of expected active components as a minimal check of the test-time complexity-control mechanism, separate from the unverified trained posterior.

## Evidence Examined

- PDF artifact: `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/paper.pdf` (6.2 MB).
- LaTeX source: `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source/main.tex`, `appendix.tex`, figures, bibliography, and ICML style files.
- Linked repository clone: `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism`.
- Artifact metadata: `source/00README.json`, which only identifies `main.tex` as the top-level source and `pdflatex` as the compiler.

Relevant paper locations:

- Symbolic model and lambda prior: `main.tex` lines 342-351.
- Claims that lambda changes posterior sparsity without retraining: `main.tex` lines 353-355 and discussion lines 591-594.
- Scaling and empirical metric claims: `main.tex` lines 364-368.
- dMRI posterior/evidence/tractography claims: `main.tex` lines 452-490.
- Broader dMRI model-family claims: `main.tex` lines 500-508.
- Software availability claim: `main.tex` lines 641-642.
- Training setup and compute: `appendix.tex` lines 220-239.
- Limited-training-data arithmetic: `appendix.tex` lines 260-263.
- Symbolic classification table: `appendix.tex` lines 294-305.
- Extended dMRI limitations and metrics: `appendix.tex` lines 440-450 and 463-500.

## Commands and Derivations

Repository/artifact inventory:

```bash
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism ls-tree -r --name-only HEAD
```

Observed output:

```text
README.md
```

```bash
sed -n '1,200p' papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism/README.md
```

Observed output:

```text
# prism

Tobe published soon.
```

The clone points to `https://github.com/mackelab/prism` at commit `500742291609efeb5201902d5c1c5c41cda3d1e7`, but the local official artifact contains only the 30-byte README. I did not pull from the network because post-release repository changes could introduce non-submission information.

Arithmetic check for simulation counts and model-space sizes:

```bash
python --version
python - <<'PY'
import math
batch=4096
steps_small=553_000
steps_large=363_000
for steps in (steps_small, steps_large):
    sims=steps*batch
    print(f'{steps} steps * {batch} batch = {sims:,} simulations = {sims/1e9:.3f} billion')
for K in (15,30,50,80,100):
    print(f'K={K}: 2^K={2**K:.6g}, 8*2^K={8*2**K:.6g}')
print('K=50 coverage vs 2^50:')
for steps in (steps_small, steps_large):
    sims=steps*batch
    print(f'  {steps}: {100*sims/(2**50):.6f}% ({sims/(2**50):.8f} fraction)')
print('K=50 coverage vs 8*2^50:')
for steps in (steps_small, steps_large):
    sims=steps*batch
    print(f'  {steps}: {100*sims/(8*2**50):.6f}% ({sims/(8*2**50):.8f} fraction)')
print('K=30 sims per 2^30 model masks:')
for steps in (steps_small, steps_large):
    sims=steps*batch
    print(f'  {steps}: {sims/(2**30):.3f}')
PY
```

Observed output:

```text
Python 3.12.12
553000 steps * 4096 batch = 2,265,088,000 simulations = 2.265 billion
363000 steps * 4096 batch = 1,486,848,000 simulations = 1.487 billion
K=15: 2^K=32768, 8*2^K=262144
K=30: 2^K=1.07374e+09, 8*2^K=8.58993e+09
K=50: 2^K=1.1259e+15, 8*2^K=9.0072e+15
K=80: 2^K=1.20893e+24, 8*2^K=9.67141e+24
K=100: 2^K=1.26765e+30, 8*2^K=1.01412e+31
K=50 coverage vs 2^50:
  553000: 0.000201% (0.00000201 fraction)
  363000: 0.000132% (0.00000132 fraction)
K=50 coverage vs 8*2^50:
  553000: 0.000025% (0.00000025 fraction)
  363000: 0.000017% (0.00000017 fraction)
K=30 sims per 2^30 model masks:
  553000: 2.110
  363000: 1.385
```

Minimal prior-control check for the symbolic prior in `main.tex` lines 349-351:

```bash
python - <<'PY'
import random, statistics
K=50
N=10000
rng=random.Random(20260424)
print('Expected and empirical active base components under symbolic prior M_i~Bern(lambda), K=50')
for lam in [0.0,0.1,0.3,0.5,0.8,1.0]:
    counts=[sum(1 for _ in range(K) if rng.random()<lam) for _ in range(N)]
    print(f'lambda={lam:.1f}: expected={K*lam:.1f}, empirical_mean={statistics.mean(counts):.2f}, sd={statistics.pstdev(counts):.2f}')
PY
```

Observed output:

```text
Expected and empirical active base components under symbolic prior M_i~Bern(lambda), K=50
lambda=0.0: expected=0.0, empirical_mean=0.00, sd=0.00
lambda=0.1: expected=5.0, empirical_mean=5.00, sd=2.16
lambda=0.3: expected=15.0, empirical_mean=15.02, sd=3.21
lambda=0.5: expected=25.0, empirical_mean=24.97, sd=3.50
lambda=0.8: expected=40.0, empirical_mean=40.01, sd=2.81
lambda=1.0: expected=50.0, empirical_mean=50.00, sd=0.00
```

## Findings

- The empirical reproduction is blocked. The paper states that code to reproduce results is available at `https://github.com/mackelab/prism` (`main.tex` lines 641-642), but the official local clone contains only `README.md` with "Tobe published soon." There are no experiment scripts, package metadata, Hydra configs, simulator code, checkpoints, datasets, or documented commands. Therefore I could not run PRISM, generate simulations through the authors' simulator, evaluate rRMSE/rKSD/SBC, reproduce Table A symbolic classification values, reproduce dMRI ESS/KSD/RMSE tables, or validate the reported runtime and tractography claims.

- The arithmetic supporting the "limited training data" narrative is mostly consistent under the paper's own stated batch size. `553k * 4096 = 2.265B` and `363k * 4096 = 1.487B`, matching the paper's "approximately 1.4--2.2 billion simulations" in `appendix.tex` line 263. Against `2^50` binary masks, this covers about `0.000132%` to `0.000201%`, i.e. order `10^-4%`, consistent with the stated tiny coverage. Against the full symbolic model count including 8 mutually exclusive noise models (`8*2^50`), coverage is eight times smaller (`0.000017%` to `0.000025%`). This distinction is not fatal to the narrative but should be stated explicitly.

- There is a wording inconsistency in the limited-training paragraph: it says "each batch corresponds to a new model" and then computes billions of simulations. The calculation actually assumes each sample within each batch can be treated as a distinct simulation/model draw. If each whole batch corresponded to only one model, the number would be hundreds of thousands, not billions.

- The paper's symbolic prior does give direct monotone prior control over expected active base components: under `M_i ~ Bern(lambda)`, `E[active base components] = K*lambda`. My small simulation for `K=50` matched this exactly within Monte Carlo error. This verifies only the prior mechanism, not the trained network's claimed posterior behavior or preservation of predictive fit when lambda is changed.

- I could not verify the central trained-posterior claims: close-to-zero rRMSE up to `2^30`, >90% top-5 accuracy across symbolic scales, dMRI evidence alignment (`R^2=0.97`), near-posterior dMRI parameter inference, runtime on H100, or tractwise correlation improvement. These are the results that carry the paper's acceptance case.

## Limitations and Blockers

- No executable implementation is present in the official repository clone.
- No requirements/environment file is present.
- No documented command exists in the README or LaTeX source for training, evaluation, or inference.
- No checkpoints or trained models are provided.
- No generated datasets or raw evaluation outputs are provided.
- No random seeds for reported experiments were found in the source.
- The paper reports H100-based training/evaluation; this workspace does not provide the authors' training setup or any runnable target to scale down.
- `pdftotext` is not installed in this environment, so I used LaTeX source as the readable paper text. This does not affect the artifact-code blocker.

## Confidence

High confidence that the official local repository artifact is insufficient to reproduce the empirical results. Medium confidence in the arithmetic and prior-control checks because they follow directly from the paper text and simple independent computations. Low confidence in the paper's central empirical claims from my role alone because the decisive evidence requires code, configs, checkpoints, and data that were not available.

## Decision Impact

Material negative reproducibility impact. The paper makes strong empirical claims about scalable amortized posterior inference and test-time complexity control, but the provided official code artifact is a placeholder. I can verify the simple model-prior arithmetic and the training-count arithmetic, but not the reported PRISM results. Under agent2's reproducibility-first standard, this should substantially downgrade confidence unless another role obtains the missing executable artifacts or independently reproduces the metrics from a complete implementation.
