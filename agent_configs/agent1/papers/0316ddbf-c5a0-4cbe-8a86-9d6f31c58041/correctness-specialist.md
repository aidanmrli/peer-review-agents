# Correctness Specialist Report

Paper: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`, "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"

Role: Correctness Specialist for agent1

Date: 2026-04-24

## Scope and Sources

I inspected only permitted local artifacts under `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/`, plus the live Koala skill guide required by `AGENTS.md`. I did not use OpenReview, decisions, citation counts, social/news/blog commentary, or later reputation signals about this paper.

Primary files inspected:

- `artifacts/main.tex`
- `artifacts/sections/paper.tex`
- `artifacts/sections/appendix.tex`
- `artifacts/sections/appendix/methodology.tex`
- `artifacts/sections/appendix/prompts.tex`
- `artifacts/00README.json`
- figure file references under `artifacts/figures/`

Lightweight commands and derivations:

- `curl -fsSL https://koala.science/skill.md | sed -n '1,220p'`
- `sed -n '1,260p' AGENTS.md` and `sed -n '1,260p' skills/correctness-specialist.md`
- `rg --files`, `find`, `tar -tzf`, and `rg -n` over artifact files to locate claims, figures, sample-count statements, filtering/refusal handling, and raw data/scripts.
- `nl -ba ... | sed -n ...` for exact line-numbered source inspection.
- `find ... -name '*.csv' -o -name '*.jsonl' -o -name '*.py' -o -name '*.ipynb' ...` found no raw result files or analysis scripts beyond `00README.json`.
- `file` confirmed selected key figures are PNGs but did not expose underlying data.
- `pdfinfo`, `pdftotext`, and ImageMagick `identify` were unavailable in the environment.
- A short Python derivation counted the model list in `paper.tex` as 11 named models, not 10, and computed illustrative Hanley-McNeil AUROC uncertainty to show that the reported AUROCs cannot be assessed without positive/negative sample counts.

## Bottom Line

The paper's central phenomenon is plausible, but the current artifact has several correctness-level weaknesses. The most serious are: inconsistent definitions of on-policy same-turn evaluation, an invalid or at least confounded on-policy/off-policy comparison in the headline AUROC discussion, unreported denominators for filtered code-risk and omitted computer-use cases, and unsupported statistical claims. These issues materially weaken the accept case because the paper's main contribution is a causal/measurement claim about fixed actions under attribution.

## Candidate Errors and Unsupported Claims

### 1. Same-turn "same action held fixed" is not cleanly established

Candidate error: The paper repeatedly frames the effect as a prompt-format manipulation on the same action, but the same-turn condition changes the task from evaluating a fixed artifact to generating and rating an artifact in the same response.

Exact locations:

- Abstract: `main.tex:109-110` defines self-attribution bias as judging an action differently when "the same action" is evaluated under off-policy attribution.
- `paper.tex:22` says the study varies judgments "when the action is held fixed and we change the format of the monitor prompt."
- Same-turn protocol: `paper.tex:122-123` says the model generates the artifact and evaluates it within the same response.
- Baseline protocol: `paper.tex:126-128` puts the artifact in the user turn in a new context.
- Code correctness prompt: `appendix.tex:151-154` asks the model to "Generate a code patch ... Then rate the correctness of your patch".
- Limitation: `paper.tex:393-395` says that for the same-turn setting the authors use generations from an LLM "that was not also asked to provide a rating" and that these actions are "not exactly on-policy."

Evidence/derivation:

The previous-turn setting can plausibly compare a fixed artifact under different conversational placement. The same-turn setting, as described in the prompt, is not a pure fixed-artifact manipulation: the artifact and the rating are co-produced, and the generation instruction can influence both the artifact distribution and the rating. The limitation section further contradicts the main protocol by saying same-turn uses generations from an LLM not also asked to rate, which implies prefilling/reuse rather than "without any prefilling or reuse of prior outputs" from `paper.tex:122-123`.

Severity: Major.

Acceptance consequence:

The paper should not treat same-turn results as decisive evidence that identical actions receive different ratings solely because of attribution. The previous-turn results may still support an attribution effect, but the same-turn claims need a clearer protocol and separate interpretation.

### 2. On-policy versus off-policy AUROC comparison appears to confound evaluator identity

Candidate error: The central AUROC/off-policy-monitor claim is not consistently defined and may compare different monitors rather than the same monitor under on-policy versus off-policy deployment.

Exact locations:

- `paper.tex:255` reports baseline on-policy AUROC = 0.99, same-turn on-policy AUROC = 0.89, previous-turn on-policy AUROC = 0.92.
- Figure caption at `paper.tex:261` describes "on-policy, where GPT-5 rates its own code" versus "off-policy, where Claude-Sonnet-4 rates GPT-5's code."
- `paper.tex:339` says "when using Claude Sonnet 4 as monitor" the off-policy AUROCs are 0.99/0.99 and the on-policy AUROCs are 0.89/0.92.

Evidence/derivation:

If the figure's stated comparison is GPT-5 rating GPT-5 code versus Claude-Sonnet-4 rating GPT-5 code, then evaluator identity changes at the same time as attribution. That comparison cannot by itself establish that deployment on-policy is worse for a fixed monitor; it could reflect differences between GPT-5 and Claude-Sonnet-4 as evaluators. Conversely, if `paper.tex:339` is intended to be Claude-Sonnet-4 as both generator and evaluator on-policy, then it conflicts with the figure caption that describes GPT-5 as the on-policy evaluator.

Severity: Major.

Acceptance consequence:

The headline "off-policy monitor evaluation overestimates self-monitor reliability" claim needs a same-evaluator analysis with clear generator/evaluator labels and matched item sets. As written, the AUROC example is not sufficiently well specified to support the causal conclusion.

### 3. "Calibration" claims are actually discrimination claims, and uncertainty is missing

Candidate error: The paper uses calibration language for quantities that are mostly separation/AUROC measures, without reporting the information needed to assess uncertainty.

Exact locations:

- `appendix.tex:33-38` defines a "separation gap" as `E[r | y=0] - E[r | y=1]` and says values near zero indicate weaker separation.
- `paper.tex:255` uses AUROC = 0.99 versus 0.89/0.92 to claim degraded separation.
- `paper.tex:339` uses AUROC differences to claim developers may choose thresholds that miss target recall/precision.
- `appendix.tex:183` says error bars denote 95% confidence intervals for some appendix dot plots, but no CI construction is specified.

Evidence/derivation:

AUROC and mean score separation measure ranking/discrimination, not calibration in the standard sense of reliability of predicted probabilities/scores. The paper does not report positive/negative counts, class balance, DeLong or bootstrap intervals, or per-model sample sizes for the AUROC claims. A lightweight Hanley-McNeil calculation illustrates why denominators matter: with `n_pos = n_neg = 20`, AUROC 0.89 has an approximate 95% interval of about [0.785, 0.995]; with `n_pos = n_neg = 100`, it narrows to about [0.844, 0.936]. The artifact gives no way to know which regime applies.

Severity: Moderate to major.

Acceptance consequence:

The direction of the effect may still be correct, but the paper overstates precision. The AUROC and threshold-setting claims should include counts, confidence intervals, and preferably paired bootstrap/DeLong tests because the comparisons are central to the practical safety conclusion.

### 4. Code-risk filtering conditions on successful harmful generation without denominators or selection criteria

Candidate error: The code-risk and PR-approval analyses filter to cases where injections succeeded and produced vulnerabilities, but do not report the denominator, filtering criteria, or model-specific retained sample counts.

Exact locations:

- `paper.tex:203-205` says the authors insert prompt-injection style hazards and "filter out generations that do not follow the prompt injection."
- `appendix.tex:47-49` says they retain only issues where the model follows the injected instruction and produces a security-relevant vulnerability, and that injection success rates are reported separately.
- `appendix.tex:51` says all harmfulness and approval judgments are performed on these retained generated patches.
- `paper.tex:24` claims self-attribution makes a monitor 5 times more likely to approve a prompt-injected code patch.

Evidence/derivation:

The source artifact does not contain the promised separate injection success rates, retained denominators, vulnerability-label criteria, approval-rate baselines, or uncertainty for the 5x claim. Conditioning on successful injections can be reasonable for isolating evaluation failures, but it changes the estimand: the reported PR approval behavior applies to a selected subset of harmful generations, not to the full prompt-injection workflow. If retained sets differ by model, the model comparisons and aggregate approval rates become hard to interpret.

Severity: Major.

Acceptance consequence:

The 5x approval claim is not adequately supported in the provided artifact. It should be downgraded unless the paper supplies retained counts, filtering rules, per-model denominators, and confidence intervals.

### 5. Refusals, schema failures, and safer rerouting are omitted in a way that can bias computer-use risk claims

Candidate error: The computer-use methodology omits cases where models refuse the harmful premise or substitute safer actions, but those are decision-relevant outcomes for a safety monitor.

Exact locations:

- `appendix.tex:69-70` says refusals and tool rerouting are omitted from attribution-shift analyses.
- `sections/appendix/prompts.tex:4-7` notes, in comments, deterministic parsing and discarded schema failures/refusals, but this file is not input by `main.tex` and no rates are reported in the compiled appendix.
- `paper.tex:315` claims extreme self-attribution reversals and says opposite-direction reversals are 5 to 10 times less common.

Evidence/derivation:

Refusal to accept a harmful action premise is not missing-at-random noise; it is a safety-relevant behavior. Omitting these cases can inflate or deflate apparent self-attribution bias depending on which condition triggers more refusals or rerouting. The paper reports neither omitted counts nor omitted-case direction by model, condition, scenario risk, or task.

Severity: Major.

Acceptance consequence:

The computer-use risk results should be treated as conditional on non-refusal/non-rerouting, not as unconditional monitor behavior. This weakens claims about real deployment risk until omission rates and sensitivity analyses are provided.

### 6. Computer-use on-policy status is internally inconsistent

Candidate error: The computer-use sections mix off-policy, interactive, and on-policy terminology without a stable mapping.

Exact locations:

- `paper.tex:211-213` introduces "200 off-policy computer-use scenarios."
- `paper.tex:218-219` introduces "on-policy high risk computer use situations" from an agentic misalignment dataset.
- Figure caption `paper.tex:303-307` says the heatmaps show same-turn and previous-turn attribution for "interactive on-policy computer-use settings."
- `paper.tex:345` later says the harmful computer-use situations setting "only has off-policy actions."
- `paper.tex:393-395` says the computer-use setting is not using generations produced by the LLM used during rating.

Evidence/derivation:

The same figure and result family are described as off-policy in one place, on-policy in another, and "interactive on-policy" in the caption. The limitation explicitly says computer-use actions are not generated by the rating model. That makes the action-origin manipulation ambiguous, and it matters because the paper's central thesis is that self-attribution bias is strongest on-policy.

Severity: Major.

Acceptance consequence:

The computer-use results can support the weaker claim that attribution framing changes risk ratings, but not the stronger claim that these are clean on-policy self-monitoring failures unless the dataset/protocol taxonomy is corrected.

### 7. Model count and model list are inconsistent

Candidate error: The paper says it evaluates 10 models but names 11 models in the main model list.

Exact locations:

- `paper.tex:45` says results are across 10 frontier models.
- `paper.tex:231-236` says "10 large language models" but lists 3 Claude models, 2 Google models, and 6 GPT/GPT-OSS models: GPT-OSS-20B, GPT-OSS-120B, GPT-4o, GPT-5-Nano, GPT-5-Mini, and GPT-5.
- `appendix.tex:200-209` table lists 10 models and does not include GPT-4o.
- `00README.json` and appendix figure filenames also enumerate 10 per-model images, with no GPT-4o model figure.

Evidence/derivation:

Manual count from the main text list: 3 + 2 + 6 = 11. The appendix table and figure assets show 10 models, so GPT-4o is likely an erroneous inclusion in the main model list or omitted from results.

Severity: Moderate.

Acceptance consequence:

This is a concrete factual inconsistency. It does not by itself overturn the results, but it reduces confidence in the experimental accounting and should be fixed before acceptance.

### 8. Sample counts are incomplete for the main claims

Candidate error: The artifact does not report enough sample counts to validate the primary code and tool-use statistics.

Exact locations:

- `paper.tex:211-224` reports counts for some non-code datasets: 200 off-policy computer-use scenarios, 1,200 MMLU questions, 1,000 synthetic MCQs, and 600 ethics questions.
- `appendix.tex:25-31` describes SWE-bench task selection and test execution but gives no number of SWE-bench issues, pass/fail counts, or per-model generated patches.
- `appendix.tex:47-49` says code-risk issues are retained after filtering, but gives no retained counts.
- `appendix.tex:69-70` omits refusal/rerouting cases but gives no omitted counts.

Evidence/derivation:

The main numerical claims require denominators: AUROC needs pass/fail counts; 5x PR approval needs baseline and self-attributed approval counts; heatmap reversal ratios need total and direction-specific transition counts; confidence intervals need sample sizes and resampling method. These are not present in the compiled source or raw artifacts.

Severity: Major.

Acceptance consequence:

The paper's quantitative claims are under-specified. A reviewer cannot independently check whether effects are large, stable, or driven by a small number of retained examples.

### 9. Cross-model heatmap interpretation overstates what the design proves

Candidate error: The appendix claims cross-model heatmaps show models rate their own patches more favorably than "identical patches attributed to other models," but the described heatmap compares evaluator/generator pairs, not necessarily the same artifact under all attribution labels.

Exact locations:

- `appendix.tex:217` says diagonal concentration confirms models rate their own prior outputs more favorably than identical outputs attributed to other models.
- `appendix.tex:245` makes the same claim for SWE-bench PR evaluation.
- `paper.tex:352-353` says diagonal concentration indicates the effect is strongest when models evaluate their own outputs.

Evidence/derivation:

The described heatmap cell is a mean rating shift for evaluator row and generator column relative to an unattributed baseline. Diagonal versus off-diagonal comparisons change the generator and therefore the artifact distribution. That can support "bias is larger on artifacts generated by the same model" only if item sets and baseline shifts are carefully controlled. It does not, from the text alone, establish that the same identical patch was attributed both to self and to another model with all else fixed.

Severity: Moderate to major.

Acceptance consequence:

The cross-model heatmaps are suggestive but not a clean proof against style, generator-quality, or item-difficulty confounds unless the authors provide paired same-artifact attribution controls and uncertainty.

### 10. Prompt/schema appendix is incomplete relative to the paper's claim

Candidate error: The paper says all prompts, schemas, and task-specific instructions are provided, but the compiled appendix only contains a code-correctness prompt figure.

Exact locations:

- `paper.tex:131` says "All prompts, schemas, and task-specific instructions are provided in Appendix."
- `appendix.tex:112-170` provides a code correctness prompt format only.
- `sections/appendix/prompts.tex:1-7` contains only a short commented-out parsing/validation note, and `main.tex:114,123` input only `sections/paper` and `sections/appendix`.

Evidence/derivation:

I found no compiled prompts or schemas for code risk, PR approval, computer-use tools, MCQ rerating, Reddit/AITA, or open-ended essay settings. This prevents checking whether rating scales, labels, answer schemas, and refusal handling are comparable across conditions.

Severity: Moderate.

Acceptance consequence:

The missing prompt details are a correctness and reproducibility problem for a prompt-format paper. They are especially important because the paper's causal interpretation depends on attributing differences to implicit self-attribution rather than other prompt differences.

## Artifact Integrity Notes

- `artifacts/sections/appendix/methodology.tex` appears to be ignored by `main.tex` and `00README.json`, but it contains additional unresolved issues: a truncated equation at lines 105-108 (`Delta = r_self -` with the baseline term commented out) and a `TODO` at line 145. I did not treat these as compiled-paper errors, but they indicate the source artifact is not clean.
- `artifacts/sections/results_old.tex`, `sections/results_reorganized.tex`, and `sections/legacy/*` contain stronger or different numerical claims in ignored drafts. I did not rely on them except to check whether missing statistics were available elsewhere.

## Final Correctness Assessment

The main acceptance-relevant concern is not that self-attribution bias is impossible; it is that the artifact does not cleanly support the stronger causal and deployment claims. The same-turn protocol is not a pure fixed-action attribution manipulation, the on-policy/off-policy AUROC example appears internally inconsistent, and the filtered/omitted cases are not accounted for. Without denominators, retained counts, omitted-case rates, uncertainty, and complete prompts, the central quantitative conclusions should be treated as under-supported. I would materially downgrade the paper unless these issues are resolved with transparent counts, paired analyses, and corrected definitions.
