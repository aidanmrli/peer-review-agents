# Correctness Specialist Report

Paper: `$V_1$: Unifying Generation and Self-Verification for Parallel Reasoners`  
Paper ID: `0a07cb4f-a3fc-42bd-988a-470a16f100e8`  
Role: Correctness Specialist  
Artifact scope checked: LaTeX source under `artifacts/source/` and official repository under `artifacts/repo/pairwise-self-verification/`.

## Commands and Inspection Log

```bash
sed -n '1,220p' skills/correctness-specialist.md
find papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source -maxdepth 2 -type f | sort
find papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification -maxdepth 3 -type f | sort
rg -n "accuracy|Pass|baseline|ablation|average|improve|percent|SWE|AIME|LiveCodeBench|CodeContests|budget|pairwise|pointwise|random" papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source -g '*.tex'
sed -n '1,340p' papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source/sections/method.tex
sed -n '1,240p' papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source/sections/appendix.tex
sed -n '1,240p' papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/README.md
sed -n '1,760p' papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/eval/verify_pairwise.py
sed -n '1,280p' papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/eval/verify_utils.py
sed -n '1,240p' papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification/config/generation.yaml
rg -n "PairRL|PointRL|DeepCoder|train|verl|DAPO|seed|std|mean" papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/repo/pairwise-self-verification papers/0a07cb4f-a3fc-42bd-988a-470a16f100e8/artifacts/source/sections -g '!*.png' -g '!*.pdf'
```

Local Python lacks `polars`, `pandas`, `pyarrow`, and `duckdb`, so I could not inspect parquet row counts directly. I therefore recomputed only numerical claims derivable from the paper text and repository scripts.

## Candidate Error 1: Main Algorithm Text Reverses Win and Loss

**Location.** `artifacts/source/sections/method.tex`, pairwise aggregation definition around the paragraph beginning "Uncertainty-Guided Score Aggregation"; Appendix Algorithm `UpdateStats`; repo `eval/verify_utils.py`.

**Claim being tested.** The paper defines an uncertainty-weighted win rate
\[
\mu_i = \frac{\sum_j w_{ij} v_{ij}}{\sum_j w_{ij}}
\]
and says `v_{ij} in {0, 0.5, 1}` denotes the outcome for `s_i`, "corresponding to a win, tie, or loss, respectively."

**Finding.** This ordering is mathematically reversed. A weighted win rate must assign `1` to a win and `0` to a loss. The appendix and code confirm the intended semantics:

```python
p_ij = 0.5 if w_sane == "tie" else (1.0 if w_sane == "A" else 0.0)
wins[i] += w * p_ij
wins[j] += w * (1.0 - p_ij)
```

If the main-text definition were followed literally, losing candidates would receive larger `mu` values and the returned ranking would invert the intended preference. This is not just notation: it is the central scoring equation in the method section.

**Severity.** Major as a specification error; likely not an implementation error because the released code and appendix use the correct direction.

**Consequence for acceptance.** The algorithm as described in the main paper is internally inconsistent. Reproducers using the main equation could implement the wrong method and obtain contradictory results.

## Candidate Error 2: Small Single-Seed Improvements Are Reported as Decisive Without Uncertainty

**Location.** `method.tex` Results and Ablations; `appendix.tex` "Experimental Runs"; repo `README.md` expected-results note.

**Claims being tested.**

- `V1-Infer` beats random pairwise verification by `+3.8%` on LCB-v6 with GPT-OSS-20B at budget `3x`.
- SWE-bench Lite pairwise verification reaches `33.3%` vs `28.3%` pointwise.
- Several benchmark gains of `4-8%` are described as demonstrating superiority.

**Evidence and derivation.**

The appendix states all base-model inference experiments are run once. The README also says the expected results are from a single seed and may vary, recommending three seeds. Yet the paper repeatedly uses causal/decisive language: "demonstrates", "consistently outperforms", "strategic pair selection ... yields more informative judgments."

For LCB-v6, the paper states `n=131`. A `+3.8%` gain is about `0.038 * 131 = 4.98`, i.e. roughly five additional problems. A conservative independent-binomial standard error for `76.3%` vs `72.5%` is:

```text
sqrt(0.763*0.237/131 + 0.725*0.275/131) = 0.054
95% interval for the difference is approximately +/- 10.6 percentage points.
```

For SWE-bench Lite, `n=300`, `33.3%` vs `28.3%` is `100/300` vs `85/300`, a 15-instance difference. Conservative independent-binomial SE is about `3.87%`, so a 95% interval for the difference is about `+/- 7.6%`. A paired test might be more powerful, but the necessary paired outcome table is not reported.

**Severity.** Major for the ablation/random-pairing claim and moderate for larger headline gains.

**Consequence for acceptance.** The paper overstates statistical certainty. Some reported deltas are plausibly within sampling/model stochasticity, especially because the same paper admits single-seed base evaluations.

## Candidate Error 3: PairRL Training Claims Are Unsupported by the Released Artifact

**Location.** `method.tex` Section "V1-PairRL"; Figure `TrainedModels_Combined_Panel_N16.png`; repo `pairwise-self-verification/`.

**Claim being tested.** The paper claims `V1-PairRL` achieves `+6.5%`, `+6.8%`, and `+7.3%` test-time gains over `V1-PointRL`, improves base Pass@1 by up to `+8.7%`, and that co-evolving training is critical.

**Finding.** The released repository is an inference/evaluation artifact, not a training artifact. Searches for PairRL, PointRL, DAPO, verl, DeepCoder training scripts, reward integration, checkpoint selection code, or validation-selection machinery found no implementation. The repo contains `eval/`, `rewards/`, dataset parquet files, and command scripts for pairwise/pointwise inference, but no train-time code matching the claimed co-evolving objective.

The paper also reports trained-model results as means over three seeds in the appendix, but the source provides only figure images, not underlying numeric tables, per-seed values, standard deviations, checkpoint identifiers, or logs. I could not recompute the stated `+6.5%`, `+6.8%`, `+7.3%`, `+8.7%`, or non-co-evolving ablation values from source tables or released result files.

**Severity.** Major.

**Consequence for acceptance.** The second half of the paper rests on empirical training claims that are not auditable from the official artifact. The claims may be true, but the evidence package does not support independent correctness checking.

## Candidate Error 4: The Released "Swiss-Parallel" Path Violates the Stated Verification-Budget Semantics

**Location.** Paper `method.tex` "Verification Budget" and "Swiss Refinement"; repo `eval/verify_pairwise.py`, function `rank_swiss_parallel`.

**Claim being tested.** `V1-Infer` is presented as an efficient sparse-comparison method using `B = 1x, 2x, 3x` pairwise calls, e.g. `N=16, budget 2x` means 32 pairwise comparisons.

**Finding.** The normal `rank_swiss` function respects the budget. However, the released `rank_swiss_parallel` implementation first calls the judge for all unordered pairs:

```python
all_pairs = [(i, j) for i in range(n) for j in range(i + 1, n)]
...
with ThreadPoolExecutor(max_workers=len(pairs_with_tags)) as executor:
    futures = {executor.submit(judge_pair, ...)}
```

Only after all `N choose 2` judge calls does it simulate the Swiss algorithm using precomputed results. For `N=16`, that is `120` pairwise LLM calls before the budgeted subset is selected, not `16`, `32`, or `48`. This function is not the default in `generation.yaml` (`coverage_strategy: min_degree`), but it is present in the official algorithm implementation and directly contradicts the "budget control" semantics if used.

**Severity.** Moderate.

**Consequence for acceptance.** The efficient-compute claim is valid only for the non-precomputed path. The artifact should clearly prevent or label the all-pairs path; otherwise reproductions may silently use a verifier with much more information than the reported budget.

## Candidate Error 5: Artifact Coverage Does Not Match Reported Benchmark Scope

**Location.** Repo `README.md` "Datasets"; actual `data/` directory; paper `method.tex` benchmark section.

**Claim being tested.** The README says all evaluation datasets are included, including `data/test_livecodebench.parquet` for LiveCodeBench-v5. The paper reports LiveCodeBench-v5, LiveCodeBench-v6, CodeContests, AIME, HMMT, and SWE-bench Lite results.

**Finding.** The local official repo contains:

```text
data/aime_2025.parquet
data/hmmt_feb_2025.parquet
data/test_code_contests.parquet
data/test_livecodebench_v6.parquet
```

It does not contain `data/test_livecodebench.parquet` for LiveCodeBench-v5, even though the command script invokes `test_livecodebench` and the README lists it. No SWE-bench Lite generation, patch, verification, or result artifact is present. The paper's Section 4 reports exact LCB-v5 and SWE-bench Lite values, but the official artifact does not include enough data to recompute them.

**Severity.** Moderate to major.

**Consequence for acceptance.** Several reported benchmark claims are not independently checkable from the provided artifact, weakening the correctness of the experimental record.

## Candidate Error 6: Exact Improvement Arithmetic Is Mostly Correct Where Stated, But Precision Is Inconsistent

**Location.** `method.tex` result paragraphs.

**Recomputations.**

- CodeContests GPT-OSS-20B: `73.33 - 66.06 = 7.27`, reported `+7.3%`. Correct after rounding.
- CodeContests Qwen3-4B-Instruct: `46.1 - 39.4 = 6.7`, reported `+6.7%`. Correct.
- SWE-bench Lite: `33.3 - 28.3 = 5.0` and `33.3 - 26.3 = 7.0`. Correct.
- Difficulty hard split: `63.9 - 40.2 = 23.7`. Correct.
- Intro says PairRL achieves `7--9%` gains over "pointwise co-training standard RL"; method gives `+6.5%`, `+6.8%`, `+7.3%` over PointRL and `+3.6%`, `+1.9%`, `+8.9%` over RL baseline with pairwise inference. The `7--9%` wording is imprecise: one of the PointRL deltas is `6.5%`, and the RL-baseline deltas are not in the `7--9%` band.

**Severity.** Minor.

**Consequence for acceptance.** Arithmetic does not reveal fabrication, but the abstract/intro summary rounds favorable results aggressively and conflates comparison targets.

## Overall Correctness Synthesis

I did not find a fatal mathematical flaw that invalidates the implemented inference algorithm. The released code's weighted win-rate update matches the intuitive method, and several simple percentage differences in the paper recompute correctly.

The serious correctness concerns are experimental rather than algebraic: the paper overclaims from single-seed/small-sample comparisons, does not expose the data or logs needed to verify several reported results, and provides no released implementation for the central PairRL training claims. The main method text also contains a reversed win/loss definition that would mislead an independent implementation. These issues materially reduce confidence in the paper's empirical conclusions, especially the strong statements about random-pairing superiority, SWE-bench generalization, and PairRL training benefits.

Score impact: substantial downgrade on correctness and empirical support. I would treat `V1-Infer` as a plausible and partially supported inference idea, but I would not accept the stronger training and broad superiority claims without underlying result tables, per-seed uncertainty, paired significance tests, and released training artifacts.
