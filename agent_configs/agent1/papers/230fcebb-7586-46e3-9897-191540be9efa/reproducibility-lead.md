# Reproducibility Lead Report

Paper ID: `230fcebb-7586-46e3-9897-191540be9efa`

Title: "Why Depth Matters in Parallelizable Sequence Models: A Lie Algebraic View"

Assigned role: Reproducibility Lead for `agent1`

Date: 2026-04-24

## Task Scope

I coordinated the internal review team for a reproducibility-first assessment of the paper's central acceptance case. The review tested whether the paper's main theoretical and empirical claims can be independently supported from the submitted paper, official artifacts, linked repository, and permitted prior literature.

The minimum reproduction target was deliberately modest: at least two independent internal roles should recover either the key empirical depth trend from executable artifacts, or independently validate the central proof logic that depth gives the claimed Lie-algebraic simulation and approximation advantage. I set the tolerance before seeing the role outcomes as follows:

- Strong reproducibility: both independent reproducers recover a central empirical result or a clearly equivalent sanity result with documented commands, and the correctness role finds no major theorem-level gap.
- Partial reproducibility: one reproducer recovers a meaningful component but another is blocked, or the empirical artifacts are plausible but missing exact result provenance.
- Weak reproducibility: neither reproducer recovers the central table/figures, or reproduction requires undocumented dependencies, hidden logs, unpinned packages, missing seeds, unavailable result files, or additional author state.
- Contradicted or unsupported claim: correctness checks find proof obligations that are not met for a central theorem.

## Claims Tested

1. Depth corresponds to a tower of Lie-algebraic extensions and allows abelian/restricted SSM stacks to simulate systems with higher derived length.
2. A restricted or abelian SSM has a simulation error lower bound controlled by the commutator/Magnus term, and depth improves local approximation order to `O(epsilon^(2^(k-1)+1))`.
3. Bounded word problems of length at most `T` can be simulated by abelian deep SSMs with at most `ceil(log2 T)+1` layers.
4. The reported experiments on finite-group word problems and `A5` rotation regression support the depth-vs-error trend.
5. The linked GitHub repository allows an independent reviewer to reproduce or audit Table 1 and the depth/MSE figures.
6. The novelty is sufficient relative to prior work on automata/state tracking, diagonal SSM expressivity, CDE/log-signature methods, and depth-length tradeoffs.

## Role Findings

### Independent Reproducer A

Outcome: partial for data generation, blocked for training and metrics.

Reproducer A created a throwaway `uv` environment and successfully generated tiny `S3` and `A5` word-problem CSVs using the official `generate_data.py` with explicit seed `123`. This partially validates the data-generation component for finite groups. However, a one-epoch eight-sample training sanity run failed after installing `requirements.txt` because the current unpinned tokenizer/datasets stack did not produce the expected `token_type_ids` column. The same pass found that `fla`, `mamba_ssm`, and `wavesAI` were still missing after `requirements.txt`, and the local environment had no `nvidia-smi`, `nvcc`, or author-specified CUDA/GCC stack.

Key commands and observations are recorded in `independent-reproducer-a.md`, including:

```bash
uv pip install --python /tmp/koala-irA-230fcebb/bin/python -r requirements.txt
/tmp/koala-irA-230fcebb/bin/python src/main.py train --group=S3 --k=4 --k_test=4 --n_layers=1 --epochs=1 --batch_size=4 --seed=1 --lr=1e-3 --model_name=transformer --max_samples=8 --train_size=0.75 --logging=False
```

Observed failure:

```text
ValueError: Column name ['token_type_ids'] not in the dataset.
```

### Independent Reproducer B

Outcome: blocked with partial code trace.

Reproducer B independently traced the released code to plausible W&B metrics for `train/max_seq_len_at_90`, `val/max_seq_len_at_90`, `train/sequence_errors`, and `val/sequence_errors`. This verifies that the repository contains a plausible implementation path for the quantities in the paper. However, the role found no raw metrics, checkpoints, W&B exports, run manifests, result CSVs, figure scripts, aggregation code, or exact seed lists sufficient to recover Table 1, the `A5` depth-vs-length figure, or the rotation MSE figure. Reproducer B also identified a train/test sample-count mismatch risk: the code default `train_size=0.99` means a generated 500000-row file is not used as 500000 training samples, and a 1000-row test file would expose only roughly 10 held-out rows under default splitting.

### Implementation Auditor

Outcome: high-severity artifact incompleteness and implementation risk.

The linked repository exists and contains relevant code at commit `e6575fae9d3aa8f32cf30254269054fbf58c87b1`. The auditor found plausible matches between the paper and code for finite-group data generation, BOS-token sequence labeling, FLA-based Transformer/GLA/DeltaProduct paths, signed Mamba flags, AUSSM integration, and hard-coded `A5` rotation matrices.

The same audit found major blockers:

- No raw results, W&B exports, checkpoints, result tables, run IDs, or plot reconstruction scripts are released.
- The hyperparameter sweeps stated in Appendix A3 are not scripted.
- `requirements.txt` is unpinned and omits major required components such as FLA, Mamba installation, and AUSSM/wavesAI.
- The local `mamba_dev` setup can use an upstream cached Mamba wheel unless a source build is forced, which risks bypassing the custom signed kernels.
- AUSSM sequence handling is under-specified and can drop the final problem token after BOS insertion through the code's multiple-of-32 crop.
- Figure 2 appears to use training-set prefix accuracy, which is weaker evidence for length generalization or exact simulation.

### Correctness Specialist

Outcome: major proof-level concerns.

The correctness role identified several decision-relevant theorem/proof gaps:

- The single-layer impossibility lemma is overstated without minimality, accessibility, nondegenerate initial state, and observability assumptions.
- The commutator-mass simulation-error lower bound does not follow from the displayed proof because the exponential map is not globally lower-Lipschitz, higher-order Magnus terms can cancel, and an operator-norm commutator bound need not be visible from the fixed initial state.
- The long-horizon error-accumulation claim is asserted from a telescoping product identity but not lower-bounded against cancellation, contraction, or unobserved directions.
- The proof that same-layer abelian affine SSM vector fields commute is false if "abelian" constrains only the `A(x)` matrices rather than the full affine vector-field algebra; scalar affine vector fields can have commuting `A` blocks and nonzero affine bracket.
- The `K`-stack simulation theorem is stated globally but the proof is local and relies on local Lie groups and local sections.
- The exponential depth-error corollary converts a leading omitted Magnus term into a simulation-error claim without the needed state/output/projection constants.
- The logarithmic-depth word-problem result does not construct an explicit exact simulator and decoder for arbitrary finite monoids.
- The state-dimension corollary stops at Witt's formula and does not tie the free Lie algebra dimension to the actual cascade SSM state dimension.

These findings directly affect the central theoretical contribution. The high-level intuition remains plausible, but the stated claims are not established as written.

### Literature Specialist

Outcome: mixed novelty with material downgrade.

The literature role found that the strongest distinctive contribution is the Lie/Magnus local depth-error formulation. The broader depth/state-tracking framing is substantially anticipated by prior work: Liu et al. on semiautomata, log-depth shortcuts, and Krohn-Rhodes structure; Merrill et al. on SSM/Transformer state-tracking limits; Hu et al. on depth-length behavior for HMM-like systems; Shakerinava et al. on `k`-layer diagonal SSM solvable-group characterization; and Walker/CDE/log-signature work on Lie brackets and structured parallel sequence models. The role also noted missing or underdeveloped baseline/citation positioning for SLiCE-style models and de-anonymized diagonal-SSM solvable-group work.

## Cross-Role Reproduction Outcome

The central empirical claim was not reproduced by two independent roles.

- Reproducer A partially reproduced the data-generation subcomponent but was blocked before even a tiny training sanity run.
- Reproducer B traced plausible metric-producing code paths but could not recover the reported table or figures because the release omits raw results, plot/aggregation scripts, and exact run manifests.
- The Implementation Auditor independently confirmed that the repository is a partial code release, not a complete reproduction package.

The central theoretical claim was also not independently validated.

- The Correctness Specialist found major theorem-level gaps in the proof chain supporting the single-layer lower bound, depth/error corollary, logarithmic-depth construction, and state-dimension claim.
- The Literature Specialist found that the paper's strongest novelty depends heavily on that same proof chain being correct, because much of the broader state-tracking/depth framing is already present in prior literature.

Under `agent1`'s rubric, the paper's core claims are weakly reproducible and partly unsupported as stated.

## Remaining Uncertainty

The released code may be capable of rerunning approximate experiments after manual dependency archaeology, older package pinning, CUDA setup, FLA installation at the stated commit, local Mamba source-build enforcement, AUSSM patching, W&B configuration, and reconstruction of a large hyperparameter grid. That uncertainty does not rescue the reproducibility record because the artifacts available to reviewers do not provide the missing state.

The theoretical results may also be repairable by narrowing statements to local-in-time simulation, full affine vector-field abelian assumptions, explicit observability/minimality conditions, and complete constructions for bounded word problems. The current submission, however, asks reviewers to accept stronger statements than the proofs establish.

## Decision Impact

The paper contains a promising and intellectually coherent idea, but its acceptance case is not reliable under a reproducibility-first standard. The empirical support is weakly reproducible, and the main theoretical novelty is undermined by proof gaps. I would recommend a weak-reject range, approximately `3.0` to `4.2`, unless other reviewers can supply an independent complete proof repair or a verified reproduction of the main figures from a pinned artifact.

