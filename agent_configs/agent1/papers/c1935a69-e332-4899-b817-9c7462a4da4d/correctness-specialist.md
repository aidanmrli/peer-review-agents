# Correctness Specialist Report

Paper: `c1935a69-e332-4899-b817-9c7462a4da4d`  
Title: "Consensus is Not Verification: Why Crowd Wisdom Strategies Fail for LLM Truthfulness"  
Role: Correctness Specialist  
Date: 2026-04-24

## Central Claim Tested

I checked the paper's central correctness claim: polling-style aggregation over LLM samples and model ensembles does not reliably improve truthfulness in verifier-absent settings because model errors and internal signals are correlated rather than truth-tracking.

The artifact has no GitHub repository and contains only the PDF, LaTeX source, bibliography/style files, and rendered figures. I therefore focused on source-level methodological, statistical, arithmetic, parsing, and conclusion-validity errors in `artifacts/source/main.tex`.

## Commands and Manual Derivations Used

```bash
sed -n '1,220p' skills/correctness-specialist.md
find papers/c1935a69-e332-4899-b817-9c7462a4da4d -maxdepth 3 -type f | sort
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '80,220p'
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '220,360p'
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '720,1025p'
rg -n "375|five benchmarks|four benchmarks|125 votes|Com2Sense omits|SP|Surprisingly Popular|inverse-SP|chance|Parsing rules" \
  papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
q_model=$((35*5 + 100*5 + 100*5 + 100*4))
echo "$q_model"
echo $((q_model*25*2*3))
echo $((q_model*25*2*2*2))
echo $(((35+100+100+100)*5*25*2*2*2))
echo $((375000/(25*2*2*2)))
awk 'BEGIN{print (3.8*500 + 3.2*500 + 2.3*175 + 1.5*400)/1575}'
```

The attempted `pdftotext` command was unavailable in this environment, so exact locations below refer to the LaTeX source.

## Findings

### 1. The reported total of 375,000 responses is arithmetically inconsistent.

Location: `main.tex:161-166`, `main.tex:175-179`, `main.tex:834-838`, `main.tex:845`.

The paper says it evaluates four verifier-absent benchmarks, with 35 HLE questions and 100 questions each for BoolQ, Com2Sense, and Predict-the-Future. It also says Com2Sense omits Gemma-3-4B in the per-model results. Under the appendix protocol, the question-model count is:

```text
35*5 + 100*5 + 100*5 + 100*4 = 1,575 question-model pairs
```

The appendix says each pair has `25` samples, `2` temperatures, `2` experiment types, and `2` prompt calls per experiment, giving:

```text
1,575 * 25 * 2 * 2 * 2 = 315,000 responses
```

If Com2Sense used all five models, the same formula gives `335,000`, still not `375,000`. The stated total implies:

```text
375,000 / (25*2*2*2) = 1,875 question-model pairs
```

That is not recoverable from the stated benchmark/model counts. The setup text also describes three response types, which would imply `236,250` responses under the 1,575-pair count. Severity: major. This is not just a typo: it makes the experimental scale, cost claim, and exact run inventory unclear.

### 2. The paper contradicts itself about Surprisingly Popular on HLE.

Location: `main.tex:288-293`, `main.tex:807-809`, `main.tex:857-861`.

The main text says inverse-SP reaches 80% on HLE, implying the standard SP signal is anti-correlated with correctness. But the appendix states: "When they do (HLE), SP yields large gains." The HLE table does not support large SP gains over `Individual Avg.`:

```text
Gemma-3-4B: 28.7 vs 27.3  = +1.4 pp
GPT-OSS-20B: 11.2 vs 10.1 = +1.1 pp
GPT-OSS-120B: 8.4 vs 11.7 = -3.3 pp
Qwen3-32B: 22.8 vs 28.2  = -5.4 pp
Qwen3-235B: 25.4 vs 21.4 = +4.0 pp
```

This is mixed, not a large-gain pattern. It also conflicts with the anti-correlated-SP interpretation. Severity: major, because SP alignment is central to the paper's explanation of why internal aggregation signals fail. The likely intended statement may have been about inverse-SP, but as written it is technically wrong.

### 3. Several conclusion claims are broader than the demonstrated evidence.

Location: `main.tex:196-203`, `main.tex:353-355`, `main.tex:687-696`, `main.tex:702-710`.

The paper evaluates five named rules, then concludes that no aggregation rule based solely on agreement, confidence, or predicted popularity can reliably scale truthfulness without a verifier. That conclusion is plausible, but it is not established as a theorem and is stronger than the experimental design supports. The experiments cover small binary subsets, two temperatures, five models, and a limited set of hand-designed signals. The source provides no proof that all possible internal signals or learned aggregators fail, and no code/data to audit whether the tested rules were correctly implemented.

Severity: major for conclusion validity. The supported claim is narrower: the evaluated rules do not consistently beat the reported single-sample baselines on the reported binary benchmarks.

### 4. The random-string control supports correlated outputs, not the claimed mechanism.

Location: `main.tex:110`, `main.tex:403-429`.

The random-string experiment can show above-chance agreement on forced-choice labels when no semantic truth exists. It does not "confirm" that the correlation stems from shared inductive biases in model weights rather than shared knowledge. Other plausible causes remain: option-label priors, tokenizer/chat-template effects, prompt wording, decoding defaults, parser behavior, and shared RLHF/instruction-following conventions. The control rules out factual knowledge as a sufficient explanation for that specific no-signal task, but it does not isolate model weights or prove the same mechanism causes benchmark errors.

Severity: moderate to major. The diagnostic is useful, but the causal interpretation is overstated.

### 5. Forecasting "chance" claims are under-specified.

Location: `main.tex:106`, `main.tex:212`, `main.tex:263-265`, `main.tex:883-887`.

The paper states that on forecasting questions "all methods perform at chance" or are "indistinguishable from chance." The reported aggregation confidence intervals in the Predict-the-Future table mostly include 50%, but the `Individual Avg.` baseline for Gemma-3-4B is `52.6% [51.3, 53.8]`, and Qwen3-235B is `51.8% [50.4, 53.1]`. If these intervals are interpreted literally, those baselines exclude 50%. If the authors intend only the final aggregation methods, the text should say so. If the claim is statistical indistinguishability, the paper should report the test or clearly define the bootstrap estimand.

Severity: moderate. This does not overturn the qualitative forecasting result, but it weakens a prominent negative-test statement.

### 6. Parsing and missing-value rules create decision-relevant bias risks.

Location: `main.tex:990-995`.

The parser lowercases responses and checks for option presence, but the source does not state what happens if both target strings occur in a response. Majority ties default to the second option (`NO` or `FALSE`). Missing SP predictions and confidence default to `0.5`, and unclear answers are excluded from vote counts. These choices can bias binary outcomes, especially in verbose or malformed model responses. Without raw outputs, malformed-output rates, or parser code, the direction and size of this bias cannot be checked.

Severity: moderate. This is a methodological correctness risk for all reported aggregation rules and for the no-signal forced-choice control.

### 7. Minor source-level inconsistencies remain.

Location: `main.tex:92`, `main.tex:161`, `main.tex:173`, `main.tex:185`, `main.tex:192`, `main.tex:845`.

The abstract says "across five benchmarks and models," while the setup lists four verifier-absent benchmarks. The HLE setup line says accuracy is far below the 50% guessing baseline and gives "(5.7%)", but the appendix HLE table reports several T=1.0 values between 8.4% and 37.4%. The figure caption and model-crowd description say five-model ensembles yield 125 votes per question, while the appendix says Com2Sense omits Gemma-3-4B. These may be editing artifacts, but they make the experimental scope harder to audit.

Severity: minor to moderate individually; collectively they reinforce that the empirical accounting is not clean.

## Checks That Did Reproduce

The temperature-flip table arithmetic is internally consistent. From `main.tex:1012-1017`:

```text
(3.8*500 + 3.2*500 + 2.3*175 + 1.5*400) / 1575 = 2.85873%, rounded to 2.9%
```

This specific claim is not a correctness problem.

## Reproducibility and Correctness Impact

The paper's broad thesis is plausible, and several reported table values directionally support "no consistent improvement." However, the correctness audit finds material source-level problems: the response count cannot be derived from the stated protocol, the SP appendix interpretation contradicts the table and main text, the random-string mechanism claim is causally over-interpreted, and parsing/default choices are insufficiently specified for a binary aggregation study.

Because no code, raw responses, selected question IDs, bootstrap scripts, parser implementation, or GitHub repository are available, I cannot determine whether these are manuscript-only errors or symptoms of deeper analysis inconsistencies. Under the agent1 reproducibility standard, that uncertainty must count against the paper.

## Final Score Impact

Correctness impact: materially negative. I would downgrade the paper from a plausible empirical story to a weak/uncertain acceptance case unless the authors supply the missing run inventory, raw outputs, aggregation/parser code, and corrected accounting. The most decision-relevant issues are major rather than fatal: they do not prove the central qualitative thesis false, but they substantially reduce confidence in the exact statistical evidence and in the strength of the conclusions.
