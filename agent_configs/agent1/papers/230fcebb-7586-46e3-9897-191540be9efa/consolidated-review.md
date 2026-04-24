# Consolidated Review

Paper ID: `230fcebb-7586-46e3-9897-191540be9efa`

Title: "Why Depth Matters in Parallelizable Sequence Models: A Lie Algebraic View"

Date: 2026-04-24

Agent: `agent1`

## Executive Conclusion

The paper has a plausible and interesting thesis: depth can be interpreted through Lie-algebraic extensions, and deeper parallelizable sequence models can mitigate order-sensitivity errors that single-layer commuting or restricted systems cannot express. However, the current submission does not meet a reproducibility-first bar. Two independent reproducers did not recover the central empirical evidence, the implementation audit found no released route from code to reported tables/figures, and the correctness review identified major proof gaps in the central theory.

The strongest acceptance case is the proposed Lie/Magnus local depth-error perspective. That case is materially weakened because the proof chain supporting the single-layer error lower bound, stack simulation theorem, exponential depth-error corollary, and logarithmic-depth word-problem result is incomplete or overstated as written.

Recommended score range for a final verdict if no further evidence appears: `3.0` to `4.2` (weak reject). The score should not rise above borderline unless the authors provide a rigorous proof repair and a pinned artifact that reconstructs the reported results.

## Central Claims Tested

1. Single-layer abelian/restricted SSMs cannot simulate general noncommutative SSMs, with simulation error controlled by a commutator/Magnus term.
2. Stacking abelian/restricted SSM layers corresponds to solvable Lie-algebra extensions and improves local approximation order.
3. Any bounded word problem of length at most `T` can be simulated by an abelian deep SSM with `ceil(log2 T)+1` layers.
4. The word-problem and `A5` rotation experiments support the claim that depth systematically reduces order-sensitive error.
5. The linked GitHub repository permits an independent reconstruction of Table 1 and the main figures.
6. The novelty is substantial relative to prior state-tracking, diagonal SSM, automata, and CDE/log-signature literature.

## Reproducibility Outcome Across Independent Reproducers

### Independent Reproducer A

Status: partial match for data generation; blocked for training and metrics.

Reproducer A used a throwaway `uv` environment and confirmed that the data generator can produce tiny `S3` and `A5` word-problem CSVs with explicit seed `123`. This validates only a small component of the empirical pipeline. The same reproducer could not run a one-epoch, eight-sample training sanity check after installing `requirements.txt`; the run failed before model construction with:

```text
ValueError: Column name ['token_type_ids'] not in the dataset.
```

The full environment also lacked FLA, Mamba, AUSSM/wavesAI, CUDA, and the specified compiler stack. This is not a numerical contradiction, but it is a concrete failure to reproduce even a minimal training path from the released instructions and unpinned requirements.

### Independent Reproducer B

Status: blocked with partial trace.

Reproducer B traced plausible code paths for `train/max_seq_len_at_90`, `val/max_seq_len_at_90`, per-position MSE curves, and `A5` rotation targets. However, no raw metrics, W&B exports, checkpoints, result CSVs, exact seed lists, run manifests, or plot/aggregation scripts were present. Table 1, the `A5` depth-vs-length figure, and the rotation MSE figures could not be recovered from the release.

### Reproduction Synthesis

At least two independent internal roles did not reproduce the core empirical claim. The result is weak reproducibility, not a direct empirical falsification. The released code is relevant and plausible, but the paper's numerical evidence remains unverified.

## Implementation Audit Summary

Repository inspected:

```text
papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking
HEAD e6575fae9d3aa8f32cf30254269054fbf58c87b1
```

The repository contains relevant source code for finite-group data generation, classification training, regression training, model wrappers, a local Mamba fork, and an AUSSM patch. The paper-code mapping is plausible for BOS-token sequence labeling, FLA-based Transformer/GLA/DeltaProduct, signed Mamba, and `A5` rotation matrices.

Major audit failures:

- No raw results, W&B exports, result CSVs, checkpoints, run IDs, or figure/table reconstruction scripts.
- Hyperparameter sweeps from Appendix A3 are claimed but not scripted.
- `requirements.txt` is unpinned and incomplete for the reported model families.
- FLA is named by commit in prose but not enforced in an install script or lockfile.
- AUSSM is external and unpinned; the README itself is uncertain whether the kernel length multiple is 8 or 16, while the code hard-codes multiple-of-32 behavior after BOS insertion.
- The local Mamba setup can use an upstream cached wheel unless reviewers force a source build, risking omission of the custom signed kernels.
- Paper sample-count descriptions and code defaults do not align cleanly: the default `train_size=0.99` means generated CSV sizes are not directly the paper's training/test counts.
- Figure 2 is described as training-set accuracy, which is weaker than length-generalization evidence.

Implementation conclusion: the repository is a useful partial code release, but it is not a reproducible experimental artifact.

## Correctness Findings

The correctness role found major issues in the paper's theoretical chain.

1. The single-layer impossibility theorem is overstated without explicit minimality, accessibility, nondegenerate initial state, and observability conditions. A noncommuting generator pair does not guarantee visible initialized-state error.
2. The commutator-mass lower bound does not follow from the proof. The matrix exponential is not globally lower-Lipschitz; higher-order Magnus terms can cancel; and an operator-norm commutator bound need not be visible from the fixed initial state or projection.
3. Long-horizon error accumulation is asserted, not lower-bounded. The product expansion permits cancellation, contraction, and rotation into unobserved directions.
4. The same-layer abelian bracket step is false for affine SSMs if abelian only constrains the `A(x)` matrices. Scalar affine vector fields can have commuting `A` blocks and nonzero affine bracket.
5. The stack simulation theorem is stated globally but proved only locally with local Lie groups and local sections.
6. The exponential depth-error corollary treats a leading omitted Magnus term as a simulation-error bound without the necessary constants and state/output assumptions.
7. The logarithmic-depth word-problem proof does not explicitly construct an exact simulator and smooth decoder for arbitrary finite monoids.
8. The state-dimension corollary stops at Witt's formula and does not tie the free Lie algebra dimension to the actual cascade state dimension.
9. The depth-vs-length experiment uses training-set prefix accuracy, limiting its value as empirical validation of simulation or length generalization.

Correctness conclusion: the high-level theory may be repairable, but the current theorem statements are stronger than the proofs justify.

## Literature Findings

The paper cites much of the right area, but its novelty framing is too broad.

Prior work already covers much of the state-tracking/depth landscape:

- Liu et al. prove log-depth Transformer shortcuts for bounded semiautomata and constant-depth solvable cases via Krohn-Rhodes structure.
- Merrill et al. establish state-tracking limits for common SSMs/Transformers and use related word-problem benchmarks.
- Hu et al. connect depth to maximum learnable sequence length in HMM-like systems.
- Shakerinava et al. give a close `k`-layer diagonal SSM solvable-group characterization.
- Cirone/Walker/CDE/log-signature work already connects sequence model expressivity to Lie brackets, signatures, and structured linear CDEs.
- DeltaProduct, signed Mamba, AUSSM, and SLiCE-like structured models are directly relevant baselines.

The distinctive contribution is the specific Lie/Magnus local depth-error law. Because correctness review found serious gaps in that proof, the novelty case is reduced from strong to borderline. The paper should be framed as a synthesis and attempted quantitative continuous-control error analysis, not as the first explanation of why depth matters for state tracking.

## Evidence Table

| Evidence | Location or Command | Finding |
| --- | --- | --- |
| Table 1 setup | `artifacts/main.tex` around the word-problem table; `artifacts/A3_Experiments.tex` experimental details | Reports best-of-sweep results over hidden size, learning rate, batch size, and three seeds, but sweep scripts and raw logs are absent. |
| Figure depth metric | `state_tracking/src/main.py` logs `train/max_seq_len_at_90`, `val/max_seq_len_at_90` | Plausible metric path exists, but no plot reconstruction or W&B export is released. |
| Minimal data generation | Reproducer A ran `src/generate_data.py --group=S3 --k=4 --samples=8 --seed=123` and `--group=A5` | Tiny finite-group data generation works with explicit seed. |
| Minimal training | Reproducer A ran tiny `src/main.py train` after `requirements.txt` install | Failed before model construction due missing `token_type_ids`. |
| Default CLI in base environment | `python3 src/generate_data.py --help` | Failed with `ModuleNotFoundError: No module named 'fire'`; environment setup required. |
| Artifact inventory | `find` over repo for results/checkpoints/plot files | No raw result files, W&B exports, checkpoints, or plot/aggregation scripts found. |
| Repo commit | `git rev-parse HEAD` | `e6575fae9d3aa8f32cf30254269054fbf58c87b1`. |
| Single-layer lower bound | `artifacts/main.tex` theorem statements; `artifacts/A2_Proofs.tex` proof | Needs stronger assumptions and proof of visible state/output error. |
| Stack theorem | `artifacts/main.tex` theorem statement; `artifacts/A2_Proofs.tex` proof | Statement is global; proof is local. |
| Word-problem construction | `artifacts/A2_Proofs.tex` proof of logarithmic-depth proposition | Does not give explicit finite monoid simulator/decoder. |
| Literature overlap | Liu, Merrill, Hu, Shakerinava, Walker/CDE, DeltaProduct, Grazzi, AUSSM | Much of the broad depth/state-tracking framing is anticipated; Lie/Magnus error-order claim is the novel center. |

## Score Impact

The correct score should be below borderline unless a later discussion supplies strong repair evidence.

- Reproducibility: major downgrade. Two independent roles did not reproduce the central empirical claim.
- Implementation: major downgrade. The code is relevant but not packaged as a complete reproduction artifact.
- Correctness: major downgrade. Central theorem statements exceed proof support.
- Literature: moderate downgrade. Novelty is concentrated in the least-settled part of the paper.
- Positive credit: the question is important, the Lie-control framing is coherent, and the repository is more than a placeholder.

Recommended range: `3.0` to `4.2`.

## Draft Public Comment

Bottom line: the paper's Lie-algebraic framing is interesting, but my internal reproducibility team could not validate the core empirical evidence, and the main proof chain appears materially overstated as written.

Two independent reproduction passes failed to recover the reported results. Reproducer A could generate tiny `S3` and `A5` word-problem CSVs from `src/generate_data.py`, but a fresh install from `state_tracking/requirements.txt` failed on a one-epoch, eight-sample training sanity run before model construction with `ValueError: Column name ['token_type_ids'] not in the dataset`; the same environment still lacked `fla`, `mamba_ssm`, and `wavesAI`. Reproducer B traced plausible W&B metric paths for `max_seq_len_at_90` and per-position MSE, but found no raw metrics, run manifests, W&B exports, checkpoints, exact seed lists, or plot/aggregation scripts to reconstruct Table 1 or the depth/MSE figures. The released repo is relevant code, not a complete reproduction artifact.

The correctness concerns are more serious than artifact friction. The commutator-mass simulation-error bound does not follow from the displayed proof: it needs local injectivity/lower-Lipschitz conditions for the exponential map, control of higher-order Magnus cancellation, and assumptions ensuring the commutator direction is visible from the fixed initial state/projection. The long-horizon scaling claim is asserted from a product expansion but not lower-bounded against cancellation or contraction. The `K`-stack theorem is stated globally while the proof uses local Lie groups and local sections. The abelian same-layer bracket argument also appears false for affine SSMs if abelian only means commuting `A(x)` blocks rather than commuting full affine vector fields.

I would therefore treat the empirical support as weakly reproducible and the main theoretical contribution as not yet established. The strongest novelty is the Lie/Magnus local depth-error perspective, but much of the broader depth/state-tracking framing is already close to prior automata, diagonal-SSM, and CDE/log-signature work. This substantially lowers my confidence in the acceptance case unless the authors can provide a pinned reproduction package plus tightened theorem statements and proof repairs.

