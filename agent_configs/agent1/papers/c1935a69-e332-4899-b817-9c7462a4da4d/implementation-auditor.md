# Implementation Auditor Report

Paper: `c1935a69-e332-4899-b817-9c7462a4da4d`, "Consensus is Not Verification: Why Crowd Wisdom Strategies Fail for LLM Truthfulness"  
Role: Implementation Auditor  
Date: 2026-04-24

## Bottom Line

The submitted artifact package does not support independent verification of the paper's empirical claims. It contains the LaTeX source, references, style files, and rendered PNG figures, but no GitHub repository, executable code, raw model outputs, datasets, benchmark construction scripts, model-call logs, parser implementation, random seeds, bootstrap/evaluation scripts, environment files, or generated result tables in machine-readable form. The paper gives useful textual descriptions of prompts, parsing defaults, benchmark filters, and result tables, but these are insufficient to audit whether the reported numbers were actually produced by the described protocol.

Acceptance impact: severe reproducibility limitation. The central empirical claims are interesting and plausible, but the artifact package is source-for-publication only, not source-for-reproduction. The absence of code/data/logs should materially reduce confidence in all quantitative claims.

## Artifact Inventory

Inspected directory: `papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/`

Present:

- `00README.json`: arXiv-style source manifest; states `main.tex` is the top-level file and `pdflatex` is the compiler.
- `main.tex`: full paper source, including methods, appendix tables, prompt templates, parser descriptions, and embedded numeric results.
- `references.bib`, `icml2026.sty`, `icml2026.bst`, `algorithm.sty`, `algorithmic.sty`, `fancyhdr.sty`: publication support files.
- `figures/*.png`: 20 rendered plots:
  - `accuracy_results.png`
  - `accuracy_comparison_hle.png`
  - `accuracy_comparison_boolq.png`
  - `accuracy_comparison_predict_the_future.png`
  - `accuracy_comparison_com2sense.png`
  - `cross_family_answer_correlation_with_truth_hle.png`
  - `cross_family_answer_correlation_with_truth_futurebench.png`
  - `cross_family_answer_correlation_with_truth_boolq.png`
  - `correlation_without_truth_0.png`
  - `correlation_without_truth_1.png`
  - `confidence_calibration_cais_hle_with_ci.png`
  - `confidence_calibration_google_boolq_with_ci.png`
  - `confidence_calibration_tasksource_com2sense_with_ci.png`
  - `confidence_calibration_kyssen_predict-the-future_with_ci.png`
  - `confidence_calibration_legend.png`
  - `agreement_vs_confidence_with_ci.png`
  - `predicted_popularity_vs_agreement_cais_hle_with_ci.png`
  - `predicted_popularity_vs_agreement_google_boolq_with_ci.png`
  - `predicted_popularity_vs_agreement_tasksource_com2sense_with_ci.png`
  - `predicted_popularity_vs_agreement_kyssen_predict-the-future_with_ci.png`
  - `predicted_popularity_legend.png`

Absent:

- No GitHub repository URL in Koala metadata (`github_urls: []`) and no repository URL in the provided task metadata.
- No `.py`, `.ipynb`, `.sh`, `.yaml`, `.csv`, `.jsonl`, `.parquet`, `.tsv`, or dataset files in the source package, apart from `00README.json`.
- No model API/inference script, sampling driver, prompt renderer, parser implementation, aggregation implementation, bootstrap CI script, plotting script, or table-generation script.
- No raw responses, response IDs, model versions/commit hashes, decoding parameters beyond temperature, seeds, cache files, failure logs, or cost/compute logs.
- No released Predict-the-Future dataset despite it being introduced by the paper.

Commands used:

```bash
rg --files skills papers/c1935a69-e332-4899-b817-9c7462a4da4d | sort
sed -n '1,220p' skills/implementation-auditor.md
cat papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/00README.json
find papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source -maxdepth 3 -type f -printf '%p\t%s bytes\n' | sort
find papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source -maxdepth 3 -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.json' -o -name '*.jsonl' -o -name '*.csv' -o -name '*.tsv' -o -name '*.parquet' -o -name '*.yaml' -o -name '*.yml' -o -name '*.txt' -o -name '*.md' -o -name '*.sh' -o -name '*.R' -o -name '*.tex' \) -printf '%p\n' | sort
rg -n "github|code|repo|artifact|data|dataset|benchmark|prompt|seed|random|temperature|model|accuracy|agreement|confidence|calibration|consensus|BoolQ|Com2Sense|Future|HLE|Table|Figure|Appendix|implementation|API|sample|bootstrap|parser|parse|json|csv" papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
```

Environment details: local shell in `/home/mila/l/lia/peer-review-agents/agent_configs/agent1`; artifact source was inspected directly from the provided paper directory. I did not modify any files outside this report.

## Paper Claim Being Tested

The implementation audit tests whether the artifact package supports the paper's core empirical claim: polling-style aggregation across repeated samples and model ensembles does not reliably improve LLM truthfulness in verifier-absent binary benchmarks, because LLM errors and outputs are correlated, confidence and predicted popularity track consensus more than correctness, and random-string controls show above-chance cross-model agreement even without truth.

Key paper locations:

- Abstract claims no consistent gains even at `25x` inference cost and claims correlated errors, confidence failure, and random-string correlation: `main.tex` lines 92-94.
- Summary contributions claim polling failure, correlated errors, endogenous signals tracking consensus, and random-string negative control: lines 118-122.
- Experimental setup describes response types, benchmarks, sampling, models, aggregation methods, evaluation, and controls: lines 155-212.
- Main result claims no method consistently outperforms single-sample baselines and forecasting is chance: lines 261-267.
- Confidence/SP claims: lines 272-293.
- Correlated-error and MATH/AIME claims: lines 297-318.
- Random-string control claims: lines 403-429.
- Consensus vs truth verification claims: lines 431-435.
- Benchmark construction, prompts, parsing, and temperature-stability appendix: lines 812-1022.

## Code Paths Inspected

There are no executable code paths to inspect. The only implementation-like material is prose in `main.tex`:

- Prompt templates at lines 949-988.
- Parser rules at lines 990-995.
- Benchmark filters and counts at lines 814-838.
- Result tables embedded as LaTeX at lines 847-895 and 1003-1020.

Because the described implementation is not present as code, I could not check actual parser behavior, aggregation equations in code, bootstrap resampling logic, model invocation settings, prompt formatting, dataset filtering, output cleaning, tie-breaking edge cases beyond prose, or plotting/table consistency.

## Paper-to-Code Matches

Only weak matches are possible because there is no code.

- The paper does disclose prompt templates for direct answers, Surprisingly Popular vote-share predictions, confidence responses, Com2Sense direct prompts, and Predict-the-Future direct prompts.
- The paper discloses parser defaults in prose: lowercasing and option-presence checks for direct answers, `YES:?\s*(\d+)` / `TRUE:?\s*(\d+)` regexes for SP predictions, defaulting missing prediction values to `0.5`, defaulting both-zero observed vote rate to `0.5`, confidence regexes, missing confidence default `0.5`, unclear answers ignored in confidence-weighted voting, and majority ties defaulting to `NO` or `FALSE`.
- The paper discloses benchmark splits/counts in a table: HLE test exactMatch yes/no subset `35`, BoolQ validation `100`, Com2Sense validation `100`, Predict-the-Future train normalized yes/no `100`.
- The paper discloses model families and temperatures in prose: Gemma-3-4B, GPT-OSS-20B, GPT-OSS-120B, Qwen3-32B, Qwen3-235B; temperatures `0.7` and `1.0` for main sampling, plus random-string figures at `0.0` and `1.0`.

These textual disclosures are useful for review, but they do not constitute runnable artifacts.

## Paper-to-Code Discrepancies and Audit Findings

1. No GitHub repository is available.

   Koala metadata has `github_urls: []`, and the artifact package does not contain a code repository. This directly blocks the required implementation audit of whether the implementation reflects the method described in the paper.

2. The raw empirical substrate is absent.

   The paper reports 375,000 model responses across datasets (lines 837-838), but the artifact includes none of those responses. It is impossible to verify response parsing, invalid/unclear rates, prompt compliance, duplicate handling, model failures, or whether all responses were sampled under the stated temperatures.

3. Benchmark construction is not independently auditable.

   The appendix gives split/filter/count summaries, but not the selected question IDs, filtered HLE subset, BoolQ/Com2Sense item IDs, or the introduced Predict-the-Future questions, labels, verification sources, and normalization script. The claim that all Predict-the-Future outcomes were manually verified and postdated model knowledge cutoffs cannot be independently checked from the artifact.

4. Parser behavior is described but not executable.

   The stated direct-answer parser checks whether target options are present after lowercasing. This can be fragile when responses contain both options, negations, explanations, or formatting deviations. The artifact does not provide the actual parser code or response corpus, so I cannot test ambiguous cases or determine whether such cases affected the reported metrics.

5. Aggregation implementations are missing.

   Majority vote, highest confidence, confidence-weighted vote, prediction-weighted vote, SP, and inverse-SP are described, but no code verifies exact formulas, tie handling beyond direct majority, normalization of confidence/predicted percentages, treatment of missing values, or whether weights were applied at the sample, question, model, or ensemble level as claimed.

6. Bootstrap confidence intervals are not reproducible.

   The paper repeatedly reports 95% bootstrap CIs over questions, but gives no bootstrap seed, number of bootstrap replicates, resampling code, stratification decisions, or source arrays. The reported CIs in tables and figures cannot be recomputed.

7. Randomness and sampling independence are unverified.

   The paper claims 25 independent samples per model at each temperature and uses random strings as a negative control. The artifact gives no random seeds, random-string generation code, random string list, decoding parameters except temperature, top-p/top-k/max-token settings, tokenizer/version settings, or model-serving backend. "Independent samples" is therefore an unverified assumption.

8. Model identity is under-specified for reproducibility.

   The models are named, but no exact Hugging Face model IDs, revisions, quantization settings, inference backend, system prompts, chat templates, tokenizer versions, or hardware/environment details are supplied. For recently released open-source models, these details can materially change outputs.

9. Figure data is not recoverable.

   The PNG figures are final rendered images only. There are no underlying CSVs or plotting scripts, so the numerical values behind correlation matrices, calibration curves, predicted-popularity plots, and accuracy bars cannot be audited except by approximate visual reading.

10. There is a scope/count ambiguity.

   The abstract says "across five benchmarks and models" (line 92), while the main verifier-absent evaluation lists four benchmarks: HLE, BoolQ, Com2Sense, and Predict-the-Future (lines 161-166). The paper later discusses MATH and AIME as verifiable-domain correlation analyses (lines 303-309). Without code/data, I cannot determine whether the "five benchmarks" phrase reflects an omitted benchmark, a counting error, or inclusion of a control/verifiable benchmark.

## Claims That Cannot Be Independently Verified

Because no code, raw data, or generated machine-readable results are included, the following claims cannot be independently verified from the artifact:

- That polling-style aggregation yields no consistent accuracy gains over single-sample baselines across the stated benchmarks, models, and temperatures.
- That the reported table values for HLE, BoolQ, Predict-the-Future, and Com2Sense are correct.
- That all methods on Predict-the-Future are indistinguishable from chance.
- That HLE accuracy is far below the 50% guessing baseline and reaches the stated values.
- That inverse-SP obtains 80% accuracy on HLE and chance-like performance elsewhere.
- That model errors are strongly correlated within and across model families.
- That cross-family answer correlations with truth match the plotted matrices.
- That random 32-character strings were generated uniformly, that 10,000 prompts were used, and that cross-model Cohen's kappa values are above chance as plotted.
- That self-reported confidence is poorly calibrated and tracks agreement rather than correctness.
- That predicted popularity tracks observed agreement rather than truth.
- That temperature changes induce only a 2.9% plurality flip rate overall, with the per-benchmark and per-model rates stated in the appendix.
- That MATH/AIME plurality-wrong and wrong-answer concentration rates are as reported.
- That there were 375,000 responses, sampled independently, with the claimed model/temperature/experiment breakdown.
- That no leakage, train/test contamination, duplicated questions, mislabeled outcomes, or post-cutoff information affected Predict-the-Future.

## Source Tables and Figures Inventory

Tables embedded in `main.tex`:

- Table `tab:benchmark_construction` (lines 817-832): benchmark construction and filtering. This is the only benchmark construction table.
- Table `tab:per_model_hle_boolq` (lines 847-871): per-model T=1.0 HLE and BoolQ results with 95% CIs.
- Table `tab:per_model_future_com2sense` (lines 873-896): per-model T=1.0 Predict-the-Future and Com2Sense results with 95% CIs.
- Table `tab:temp_stability` (lines 1003-1020): plurality answer flip rates between T=0.7 and T=1.0.

Figures embedded in `main.tex`:

- Figure `fig:ensemble_accuracy` (lines 182-187): ensemble aggregation accuracy, using `figures/accuracy_results.png`.
- Figure `fig:cross_family_answer_correlation_with_truth` (lines 232-259): HLE, Predict-the-Future, and BoolQ answer/truth correlations.
- Figure `fig:correlation_without_truth` (lines 321-339): random-string Cohen's kappa at temperatures 0 and 1.
- Figure `fig:confidence_calibration` (lines 358-397): reliability diagrams for HLE, BoolQ, Com2Sense, and Predict-the-Future.
- Figure `fig:agreement_vs_confidence` (lines 438-442): confidence versus agreement.
- Figure `fig:predicted_popularity_vs_agreement` (lines 557-598): predicted vote share versus observed agreement for four datasets.
- Figure `fig:per_model_accuracy_main` (lines 898-920): per-model aggregation accuracy for HLE and BoolQ.
- Figure `fig:per_model_accuracy_aux` (lines 922-944): per-model aggregation accuracy for Predict-the-Future and Com2Sense.

Commented-out figures/tables are present in the LaTeX around lines 222-228, 474-518, and related commented sections, but those are not part of the rendered artifact.

## Included Benchmark Construction Details

Included only as prose/table:

- HLE: `test`, `exactMatch & yes/no only`, answer format `YES/NO`, `35` questions.
- BoolQ: `validation`, boolean labels mapped to `TRUE/FALSE`, `100` questions.
- Com2Sense: `validation`, labels mapped to `TRUE/FALSE`, `100` questions; Gemma-3-4B omitted in per-model Com2Sense results.
- Predict-the-Future: `train`, normalized yes/no, `100` questions; described as introduced in the paper, manually verified, and postdating model knowledge cutoffs.
- Sampling: 25 samples per question/model at T=0.7 and T=1.0 for each experiment type; direct-answer and prediction/confidence prompts; claimed total 375,000 responses.

Missing construction details:

- Exact question IDs and labels for all four verifier-absent benchmarks.
- Full Predict-the-Future dataset, event sources, event resolution dates, label verification notes, and cutoff reasoning.
- HLE binary subset selection script.
- BoolQ/Com2Sense subset sampling procedure for the 100 selected validation questions.
- Any deduplication, filtering, or contamination checks.
- The MATH/AIME problem IDs used for the verifiable-domain error-concentration analysis.

## Severity for Acceptance Decision

Severity: high.

This is not a minor packaging issue. The paper's acceptance case rests on quantitative empirical claims about model sampling, aggregation, calibration, correlations, and benchmark construction. The artifact package provides only the manuscript source and final rendered figures. With no repository, no code, no raw responses, no dataset release for the new benchmark, and no scripts to regenerate tables/figures, reviewers cannot independently validate the central conclusions or identify implementation bugs. The appropriate conclusion is weak artifact support: the paper may be scientifically valuable, but its empirical claims are not reproducible from the submitted artifacts.

