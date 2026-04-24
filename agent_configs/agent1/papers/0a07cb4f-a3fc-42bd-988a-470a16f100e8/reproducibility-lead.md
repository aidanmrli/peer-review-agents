# Reproducibility Lead Report

Paper ID: `0a07cb4f-a3fc-42bd-988a-470a16f100e8`

Title: `$V_1$: Unifying Generation and Self-Verification for Parallel Reasoners`

Assigned role: Reproducibility Lead

Date: 2026-04-24

## Task Scope

I coordinated the internal review protocol for `agent1`, identified the central claims, compared the two independent reproducer outcomes, and determined how the reproduction evidence should affect a public Koala comment.

## Claims Tested

1. `V_1`-Infer: pairwise same-model self-verification with a Swiss/min-degree tournament improves candidate selection over pointwise verification and RSA-style aggregation on code/math/SWE tasks.
2. `V_1`-PairRL: online co-evolving RL trains a single model as both generator and pairwise self-verifier, improving test-time scaling and base Pass@1 over RL and PointRL baselines.
3. Artifact reproducibility: the official repository, data, scripts, and paper source allow independent audit of the reported metrics.

Minimum reproduction target set before outcomes: recover at least one reported benchmark metric or, if compute/dependencies block this, independently validate the smallest meaningful unit of the ranking/evaluation pipeline and document blockers.

## Evidence Examined

- Paper source under `papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source/`.
- Official repository under `papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/`.
- Repository commit `be5595566a7f19ae785f1695d05cff1e0c7db834`.
- Koala comments on this paper as of the review.
- Internal reports:
  - `independent-reproducer-a.md`
  - `independent-reproducer-b.md`
  - `implementation-auditor.md`
  - `correctness-specialist.md`
  - `literature-specialist.md`

## Role Findings

Independent Reproducer A attempted a documented math run on a minimal setting and reproduced only the local margin-weighted aggregation arithmetic on a synthetic tournament. The documented run blocked before dataset/model execution because the local data files were Git LFS pointers and the active environment lacked required dependencies, starting with `huggingface_hub`.

Independent Reproducer B used an independent route: code-level tracing plus a dependency-free synthetic sanity check of the Swiss/pairwise aggregation path. This supported the qualitative mechanism when pairwise ratings contain information that pointwise saturated scores lose, but did not reproduce any benchmark metric. It independently found LFS pointer data, no local `git lfs`, missing dependencies, and absent training/checkpoint/result artifacts.

The Implementation Auditor found that the repository is best characterized as a `V_1`-Infer evaluation harness. It contains pairwise/pointwise inference and grading scripts, but no PairRL, PointRL, generation-only RL training code, DeepCoder training pipeline, trained checkpoints, raw result parquets, cached generations, or pinned local environment. The local `data/*.parquet` files are pointer files, and `test_livecodebench.parquet` for LCB-v5 is absent in the checkout.

The Correctness Specialist found no fatal flaw in the implemented weighted-win-rate aggregation, but identified a major paper-specification error: the main method text reverses win/loss coding for `v_ij`; the appendix and code use the intended semantics. The report also found that base-model inference results are single-seed, several small deltas are overstated without paired tests or uncertainty, and the PairRL results are not auditable from the released artifact.

The Literature Specialist concluded that the novelty is real but narrower than the broad framing. Same-model pairwise self-verification with sparse tournament scheduling is a useful integration, but pairwise ranking, best-of-N selection, verifier training, and unified generator-verifier RL have substantial prior art. The broad "unifying generation and self-verification" framing is stronger than the evidence supports.

## Reproducibility Outcome

The core `V_1`-Infer mechanism is partially reproducible at the algorithmic sanity-check level. Both independent reproducers validated the qualitative behavior of the margin-weighted pairwise ranking logic on synthetic examples or arithmetic traces.

The central empirical claims were not reproduced by either independent reproducer. No role recovered a reported benchmark accuracy, SWE-bench result, RSA comparison, trained-model score, or PairRL gain.

The `V_1`-PairRL claim has weak reproducibility from the artifact. The paper describes an important training contribution, but the visible official repository does not contain the training implementation, checkpoints, logs, raw metrics, or per-seed results needed to audit it.

Practical reproducibility classification:

- `V_1`-Infer algorithm mechanics: partial reproducibility.
- Reported base-model benchmark improvements: weak reproducibility.
- `V_1`-PairRL training claims: weak reproducibility.
- Literature framing: partially supported, with novelty inflation.

## Commands and Checks Relied On

Representative commands from the role reports:

```bash
git -C papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification rev-parse HEAD
for f in papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/data/*.parquet; do sed -n '1,8p' "$f"; done
rg -n "PairRL|PointRL|DeepCoder|train|verl|DAPO|checkpoint" papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification
bash -n eval/scripts/run_e2e_pairwise.sh eval/scripts/run_e2e_pairwise_math.sh eval/scripts/run_e2e_pointwise.sh eval/scripts/run_e2e_pointwise_math.sh
python3 -m py_compile eval/verify_utils.py eval/verify_pairwise.py eval/eval_e2e_pairwise.py eval/eval_e2e_pointwise.py eval/eval_results_parallel.py eval/utils.py rewards/*.py
```

Important observed evidence:

- `git lfs` was unavailable locally.
- The checked-out dataset files contained `version https://git-lfs.github.com/spec/v1` pointer records.
- The active Python environment lacked the documented runtime stack.
- The official README says expected results are single-seed and recommends three seeds for stability.

## Decision Impact

The paper should receive credit for a plausible, inspectable inference-time idea and for releasing a partial evaluation harness. It should not receive full reproducibility credit for the headline empirical claims or the training contribution. The missing PairRL training artifact, unresolved datasets, missing raw results, and single-seed/small-sample uncertainty materially weaken the acceptance case.

Recommended public stance: a confident, decision-relevant top-level comment emphasizing that the reported result is not reproducible from the stated artifact package by two independent internal reproducers, while separating this from the narrower point that the `V_1`-Infer aggregation logic itself appears coherent.

Recommended score impact for a later verdict: substantial downgrade on reproducibility and empirical support; likely weak-reject to borderline unless subsequent discussion or author artifact updates establish the missing training/results evidence.

## Remaining Uncertainty

The reported results may still be correct if rerun in the authors' Modal/H100 environment with the intended Hugging Face datasets, SGLang setup, cached generations, and unreleased training infrastructure. That uncertainty does not rescue reproducibility for peer review: the artifact available to reviewers does not currently allow two independent roles to reproduce the central empirical or training claims.
