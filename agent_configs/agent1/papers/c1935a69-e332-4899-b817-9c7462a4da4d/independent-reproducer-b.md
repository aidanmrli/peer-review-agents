# Independent Reproducer B Report

Paper: `c1935a69-e332-4899-b817-9c7462a4da4d`, "Consensus is Not Verification: Why Crowd Wisdom Strategies Fail for LLM Truthfulness"

Role: Independent Reproducer B

## Claim Attempted

I attempted to independently validate the paper's central empirical claim that polling-style aggregation does not reliably improve LLM truthfulness in verifier-absent binary benchmarks, even with substantially increased inference-time sampling. My route did not rerun model inference because the artifact directory contains the paper source and figures but no generated response data, benchmark files, scripts, random seeds, or linked GitHub repository.

Instead, I checked whether the reported evidence is internally reproducible from the paper's own benchmark construction, sampling protocol, prompt/parsing details, and confidence interval statements. This is a distinct route based on arithmetic consistency and statistical interpretability of the reported tables.

## Source and Artifact Scope

- Paper source inspected: `papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex`
- Figures present: `papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/figures/*.png`
- No linked GitHub repository was available in the local paper artifacts.
- No raw model outputs, dataset subsets, parsing outputs, bootstrap scripts, or aggregation code were available locally.

## Independent Route Used

I used three checks:

1. Recompute the paper's response-count claim from the appendix benchmark table and sampling protocol.
2. Check whether the reported bootstrap confidence intervals are compatible with the paper's statement that 95% CIs are obtained by resampling questions.
3. Inspect prompt/parsing details for hidden choices that affect reproducibility of the aggregation results.

## Commands and Trace

I first inspected the role instructions and source files without reading Independent Reproducer A's report:

```bash
pwd && rg --files
sed -n '1,220p' skills/independent-reproducer-b.md
sed -n '1,260p' papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
sed -n '260,620p' papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
sed -n '620,1045p' papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
rg -n "375,000|benchmark-wise|Appendix|Table|tabular|Predict|random|string|confidence|bootstrap|accuracy|chance|5\\.7|100|25|125|correlation|Fleiss|kappa|Com2Sense|BoolQ|HLE" papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
identify papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/figures/*.png 2>/dev/null || file papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/figures/*.png
```

I then recomputed the response-count and CI sanity checks:

```bash
python - <<'PY'
# Recompute protocol-implied response counts and simple CI sanity checks from the paper source.
benchmarks = {
    'HLE': {'questions': 35, 'models': 5},
    'BoolQ': {'questions': 100, 'models': 5},
    'Com2Sense': {'questions': 100, 'models': 4}, # appendix: Com2Sense omits Gemma-3-4B
    'Predict-the-Future': {'questions': 100, 'models': 5},
}
q_model = sum(v['questions'] * v['models'] for v in benchmarks.values())
print('question_model_tuples', q_model)
print('setup_direct_count_25_samples_x_2_temps_x_3_response_types', q_model * 25 * 2 * 3)
print('appendix_count_25_samples_x_2_temps_x_2_experiments_x_2_prompt_types', q_model * 25 * 2 * 2 * 2)
print('if_Com2Sense_had_5_models_appendix_formula', (35*5 + 100*5 + 100*5 + 100*5) * 25 * 2 * 2 * 2)
print('temperature_table_total', 500+500+175+400)
# CI sanity: use binomial approximation as a conservative reference for question-level binary indicators.
import math
for name, p, n, reported_lo, reported_hi in [
    ('HLE Gemma individual avg', 0.273, 35, 0.253, 0.294),
    ('BoolQ Gemma individual avg', 0.611, 100, 0.597, 0.624),
    ('Predict Future Gemma individual avg', 0.526, 100, 0.513, 0.538),
]:
    half = 1.96 * math.sqrt(p*(1-p)/n)
    print(name, 'question_level_binomial_95_halfwidth_pct', round(100*half, 1), 'reported_halfwidth_pct', round(100*((reported_hi-reported_lo)/2), 1))
PY
```

Observed output:

```text
question_model_tuples 1575
setup_direct_count_25_samples_x_2_temps_x_3_response_types 236250
appendix_count_25_samples_x_2_temps_x_2_experiments_x_2_prompt_types 315000
if_Com2Sense_had_5_models_appendix_formula 335000
temperature_table_total 1575
HLE Gemma individual avg question_level_binomial_95_halfwidth_pct 14.8 reported_halfwidth_pct 2.0
BoolQ Gemma individual avg question_level_binomial_95_halfwidth_pct 9.6 reported_halfwidth_pct 1.4
Predict Future Gemma individual avg question_level_binomial_95_halfwidth_pct 9.8 reported_halfwidth_pct 1.3
```

## Findings

### 1. The reported total of 375,000 model responses is not reproducible from the stated benchmark table and sampling protocol.

The setup states 25 samples per model at each of two temperatures and three response types: direct answer, confidence, and predicted vote share. Using the appendix table and the paper's note that Com2Sense omits Gemma-3-4B, the question-model total is:

- HLE: 35 questions x 5 models = 175
- BoolQ: 100 x 5 = 500
- Predict-the-Future: 100 x 5 = 500
- Com2Sense: 100 x 4 = 400
- Total: 1,575 question-model tuples

Under the setup description, `1,575 x 25 x 2 x 3 = 236,250` responses, not 375,000.

The appendix gives a different protocol: two experiment types, each issuing direct-answer plus prediction/confidence prompts, yielding `1,575 x 25 x 2 x 2 x 2 = 315,000` responses, again not 375,000.

Even if Com2Sense incorrectly used all five models, the appendix-style count would be `335,000`, still not 375,000.

This blocks independent reconstruction of the experimental scale and makes it unclear which runs underlie the reported figures.

### 2. The reported confidence intervals appear inconsistent with "resampling questions."

The paper states that accuracy uses 95% bootstrap CIs obtained by resampling questions. However, the "Individual Avg." intervals in Appendix Tables 2 and 3 are far narrower than question-level resampling should allow for 35 or 100 questions.

Examples:

- HLE Gemma individual average: reported 27.3% [25.3, 29.4], half-width about 2.0 percentage points. A simple question-level binomial reference at 35 questions gives about 14.8 percentage points.
- BoolQ Gemma individual average: reported 61.1% [59.7, 62.4], half-width about 1.4 percentage points. A 100-question reference gives about 9.6 percentage points.
- Predict-the-Future Gemma individual average: reported 52.6% [51.3, 53.8], half-width about 1.3 percentage points. A 100-question reference gives about 9.8 percentage points.

These narrow intervals are much more consistent with treating individual samples as independent than with resampling questions. If so, the uncertainty estimates understate question-level uncertainty and are not directly comparable to the aggregation CIs that appear to be question-level.

This matters for the central claim because "no consistent improvement" is assessed visually and statistically through overlapping error bars. If the CIs mix sample-level and question-level uncertainty, the interpretability of the empirical comparison is weakened.

### 3. Prompt and parsing choices introduce undocumented degrees of freedom.

The appendix documents important defaults:

- Direct-answer parsing lowercases responses and checks for target option presence.
- Missing direct answers become `unclear` and are excluded from vote counts.
- Majority ties default to the second option, NO or FALSE.
- Missing SP predictions default to 0.5.
- Missing confidence defaults to 0.5.
- Missing confidence answers become `unclear` and are ignored in confidence-weighted voting.

These defaults are decision-relevant for binary datasets, especially when models emit malformed or verbose answers. Without raw outputs or code, I cannot verify how often these defaults fire or whether they bias methods toward the second option. This is a reproducibility limitation rather than direct disconfirmation.

## Observed Result

Blocked/partial reproduction.

The paper source supports the qualitative existence of the intended experiment: four verifier-absent binary benchmarks, five models except Com2Sense with four, 25 samples, two temperatures, and aggregation via majority, confidence, predicted popularity, and SP. However, the supplied artifacts do not allow independent recomputation of any model-level result. More importantly, two paper-internal checks fail:

1. The total response count cannot be derived from the stated protocol.
2. Several reported CIs are not compatible with the stated question-level bootstrap procedure.

These issues do not prove that the central empirical conclusion is false, but they materially block reproducibility of the evidence supporting it.

## Match / Partial Match / Mismatch / Blocked

Outcome: **Blocked with partial support**.

Partial support:

- The benchmark construction table and temperature-stability table are internally consistent on the 1,575 question-model tuple count.
- The appendix per-model tables qualitatively show many cases where aggregation fails or degrades performance, which is directionally consistent with the claim.

Blocked:

- No raw data, code, seeds, generated responses, bootstrap implementation, or aggregation scripts are present.
- The paper's own arithmetic does not reproduce the headline 375,000 response count.
- The reported "bootstrap over questions" intervals appear inconsistent with question-level resampling.

## Reproduction Outcome for Independent Reproducer B

I cannot independently validate the central empirical claim to the paper's stated standard from the provided artifacts. The source provides figures and tables but not enough machinery to reproduce them, and internal numerical inconsistencies reduce confidence in the reported experimental accounting and uncertainty estimates.

Score impact from this role: materially negative for reproducibility. I would treat the paper's high-level thesis as plausible but not independently reproduced. The required downgrade is substantial if the review standard demands artifact-level or statistically auditable reproduction.

## Agreement With Reproducer A

Not checked. I intentionally did not read Independent Reproducer A's report, per assignment. This field should be reconciled only after both independent passes are complete.
