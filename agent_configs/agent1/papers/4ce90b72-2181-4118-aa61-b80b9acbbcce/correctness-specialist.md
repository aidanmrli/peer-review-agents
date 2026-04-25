# Correctness Specialist Report

## Paper

- Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`
- Title: `Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes`
- Assigned role: Correctness Specialist
- Task scope: method equations, objective definitions, sparsity/BatchTopK/Dual-K claims, delta loss, contrastive pairing, causal steering/ablation evaluation, grader comparisons, null experiment, hyperparameters, and metric interpretation.

## Evidence Examined

- Local source: `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex`
- Local bibliography: `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.bib`
- Local figures directory: `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/figures/`
- Local repository directory: `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/repos/`, which contained no files beyond the directory itself during this pass.
- No OpenReview material, decisions, citation counts, social media, or later outcome signals were used.

## Commands and Checks

```bash
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce -maxdepth 3 -type f | sort
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/repos -maxdepth 2 -type f -o -type d | sort
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '260,430p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '430,660p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '660,870p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '870,1165p'
rg -n "ablat|causal validation|causally|false positive|10/10|right tail|relative decoder norm|GPT-5.2|grader|best reported|without finetuning data|52.5|confidence|interval|std|variance" papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex
python3 - <<'PY'
rows = [
('LLaMA EM (Extreme Sports)','Delta',80.46,8.1),('LLaMA EM (Extreme Sports)','TopK-200',79.29,16.1),('LLaMA EM (Extreme Sports)','TopK-400',81.64,33.5),('LLaMA EM (Extreme Sports)','DSF',80.07,17.2),
('LLaMA EM (Risky Finance)','Delta',80.46,7.6),('LLaMA EM (Risky Finance)','TopK-200',79.29,14.2),('LLaMA EM (Risky Finance)','TopK-400',81.64,30.8),('LLaMA EM (Risky Finance)','DSF',80.07,18.2),
('LLaMA EM (Bad Medical)','Delta',80.07,9.4),('LLaMA EM (Bad Medical)','TopK-200',79.29,15.4),('LLaMA EM (Bad Medical)','TopK-400',81.64,30.4),('LLaMA EM (Bad Medical)','DSF',80.07,17.6),
('Qwen EM (Extreme Sports)','Delta',80.07,22.4),('Qwen EM (Extreme Sports)','TopK-200',79.68,13.0),('Qwen EM (Extreme Sports)','TopK-400',80.07,64.3),('Qwen EM (Extreme Sports)','DSF',80.07,23.2),
('Qwen Subliminal','Delta',76.17,64.2),('Qwen Subliminal','TopK-200',79.29,8.3),('Qwen Subliminal','TopK-400',80.07,57.8),('Qwen Subliminal','DSF',80.07,9.5),
('Gemma Taboo (Gold)','Delta',76.17,16.4),('Gemma Taboo (Gold)','TopK-200',74.21,20.4),('Gemma Taboo (Gold)','TopK-400',79.29,12.0),('Gemma Taboo (Gold)','DSF',75.39,18.4),
('LLaMA SDF (Cake Bake)','Delta',72.65,55.8),('LLaMA SDF (Cake Bake)','TopK-200',69.53,57.1),('LLaMA SDF (Cake Bake)','TopK-400',77.73,54.0),('LLaMA SDF (Cake Bake)','DSF',72.65,51.9),
('LLaMA SDF (Abortion)','Delta',72.26,56.0),('LLaMA SDF (Abortion)','TopK-200',69.53,56.8),('LLaMA SDF (Abortion)','TopK-400',77.73,53.6),('LLaMA SDF (Abortion)','DSF',73.04,53.3),
]
from collections import defaultdict
by=defaultdict(list)
for r in rows: by[r[0]].append(r)
for org, vals in by.items():
    delta=[v for v in vals if v[1]=='Delta'][0]
    best_dead=min(vals, key=lambda v:v[3])
    best_ev=max(vals, key=lambda v:v[2])
    print(f'{org}: delta dead {delta[3]} vs best {best_dead[1]} {best_dead[3]} (diff {delta[3]-best_dead[3]:+.1f}); delta EV {delta[2]} vs best {best_ev[1]} {best_ev[2]} (diff {delta[2]-best_ev[2]:+.2f})')
PY
```

Metric recomputation output:

```text
LLaMA EM (Extreme Sports): delta dead 8.1 vs best Delta 8.1 (diff +0.0); delta EV 80.46 vs best TopK-400 81.64 (diff -1.18)
LLaMA EM (Risky Finance): delta dead 7.6 vs best Delta 7.6 (diff +0.0); delta EV 80.46 vs best TopK-400 81.64 (diff -1.18)
LLaMA EM (Bad Medical): delta dead 9.4 vs best Delta 9.4 (diff +0.0); delta EV 80.07 vs best TopK-400 81.64 (diff -1.57)
Qwen EM (Extreme Sports): delta dead 22.4 vs best TopK-200 13.0 (diff +9.4); delta EV 80.07 vs best Delta 80.07 (diff +0.00)
Qwen Subliminal: delta dead 64.2 vs best TopK-200 8.3 (diff +55.9); delta EV 76.17 vs best TopK-400 80.07 (diff -3.90)
Gemma Taboo (Gold): delta dead 16.4 vs best TopK-400 12.0 (diff +4.4); delta EV 76.17 vs best TopK-400 79.29 (diff -3.12)
LLaMA SDF (Cake Bake): delta dead 55.8 vs best DSF 51.9 (diff +3.9); delta EV 72.65 vs best TopK-400 77.73 (diff -5.08)
LLaMA SDF (Abortion): delta dead 56.0 vs best DSF 53.3 (diff +2.7); delta EV 72.26 vs best TopK-400 77.73 (diff -5.47)
```

## Findings

### 1. Delta loss is mathematically redundant in its first definition and ill-defined for unmatched inputs

- Candidate error: The paper presents the delta loss as a first-class signal for fine-tuning differences, but the unmasked version is a function of the two reconstruction residuals when the same sparse code is used for both reconstructions. It also states that `Delta = b - a` "does not require" matched inputs, which is not technically valid if the goal is to isolate model-induced differences.
- Exact locations: lines 311-316 define `a`, `b`, and `Delta = b - a` and claim matched inputs are unnecessary; lines 318-333 define reconstructions and `L_Delta`; lines 348-357 introduce contrastive matched text pairs; line 357 says training mixes contrastive pairs and unpaired activations.
- Evidence/derivation: With `e_a = a - W_base z` and `e_b = b - W_ft z`, the loss in lines 328-333 is
  `||Delta - (W_ft - W_base)z||^2 = ||(b-a) - (W_ft z - W_base z)||^2 = ||e_b - e_a||^2`.
  Therefore, before the shared-feature mask is introduced, this loss does not directly identify a new fine-tuning direction; it penalizes disagreement between the two reconstruction residuals. If `a` and `b` are sampled from unrelated inputs, `b-a` is dominated by semantic/content variation between examples, not by the causal effect of fine-tuning. Matched activations on the same input can define a meaningful model difference, but the manuscript explicitly says matching is unnecessary and later says unpaired activations are mixed in.
- Severity: Major.
- Consequence for acceptance: The central objective is under-justified. The paper can still be empirically useful, but the claim that the delta loss directly prioritizes fine-tuning-induced directions is not established by the equations as written.
- Confidence: High.

### 2. Relative decoder norm selection conflicts with the paper's own definition

- Candidate error: The paper defines `R_base_i` so that the right tail denotes base-specific features, then repeatedly says it selects the "right tail" to find fine-tuning-induced or non-shared latents.
- Exact locations: lines 293-301 define `R_base_i = ||d_base_i|| / (||d_base_i|| + ||d_ft_i||)` and state that values near 1 are base-specific while values near 0.5 are shared. Lines 538-540 select the top-3 non-shared latents from the right tail of the relative decoder norm distribution. Lines 647 and 707 again describe salient right-tail latents. Lines 813-816 use the same criterion in the false-positive discussion.
- Evidence/derivation: Under the definition in lines 293-301, a finetuned-specific feature should have small `R_base_i` and large `1 - R_base_i`, not a large `R_base_i`. If the authors instead use a finetuned-relative norm, that variable is not defined. This makes the latent ranking rule ambiguous and internally inconsistent.
- Severity: Major.
- Consequence for acceptance: The main evaluation pipeline depends on this ranking rule. Without a corrected definition, the selection of "causal" latents is not technically auditable.
- Confidence: High.

### 3. Dual-K/BatchTopK objective is underspecified and the shared-masking guarantee is overstated

- Candidate error: The paper claims Dual-K allocation and shared-feature masking ensure difference signals cannot be absorbed by shared features, but the optimization problem does not enforce that guarantee.
- Exact locations: lines 359-380 define the shared/non-shared partition and masked delta loss; lines 382-390 define the full objective; lines 884-907 give `Delta Lambda = 0.005`, `Base Sparsity = 200`, `Shared k Multiplier = 2.0`, `Shared Features Fraction = 20%`, and `AuxK Coefficient = 1/32`.
- Evidence/derivation: The delta term masks shared latents, but the reconstruction term still reconstructs `a` and `b` using the full code. Since the delta term is weighted by only `0.005`, model-specific shared decoders can still encode base/finetuned differences through reconstruction if that improves the dominant loss. Thus line 380's statement that shared features "cannot absorb fine-tuning-specific differences" is stronger than the objective supports. The sparsity mechanism is also not fully specified: the paper says `K_Delta = alpha * K_shared` with `alpha < 1`, but the appendix gives `AuxK Coefficient = 1/32` and a shared multiplier without defining the actual non-shared `K_Delta` used, whether BatchTopK is applied per partition or globally, or whether `K` is per token or over the batch.
- Severity: Major.
- Consequence for acceptance: The method may be implementable by the authors, but the paper's objective is not precise enough to verify the claimed mechanism or reproduce the allocation logic.
- Confidence: High.

### 4. Contrastive pairing can confound fine-tuning effects with generated-content differences

- Candidate error: The contrastive data construction is described as task-agnostic and causally downstream of fine-tuning, but it can encode surface content differences from model-generated responses.
- Exact locations: lines 348-357 construct prompts, generate `y_base` and `y_ft`, concatenate each response to the prompt, and claim activation differences concentrate on regions causally downstream of the fine-tuning objective. Lines 505-517 describe the training data mixture, including fine-tuning data by default and 200,000 contrastive prompts. Lines 1103-1121 claim removing fine-tuning data does not degrade recovery, based on two settings.
- Evidence/derivation: If the finetuned model response contains domain-specific text and the base response does not, a feature trained on `x || y_ft` versus `x || y_base` can recover lexical or response-distribution artifacts rather than a model-internal causal mechanism. Passing each concatenated input through both models helps define matched activations for a fixed text, but the text itself was selected/generated by model behavior. The claim that the resulting activation differences are causally downstream of the finetuning objective is plausible but not proved by the setup.
- Severity: Moderate to major.
- Consequence for acceptance: The paper's "task-agnostic" and causal interpretation should be narrowed unless there are held-out controls separating generated-content cues from model-difference directions.
- Confidence: Medium-high.

### 5. Steering results show intervention effects, not that latents are causally responsible

- Candidate error: The paper repeatedly uses "causal", "causally responsible", and "causal validation", but the evaluation described is mostly positive/negative steering plus qualitative max-activation inspection. That is evidence of sufficiency under intervention, not evidence that the latent is necessary for the natural behavior.
- Exact locations: abstract line 191; introduction/contributions lines 232-240; evaluation methodology lines 538-558; results overview line 596; EM causal role claims lines 709-716; discussion lines 813-816 and 849; appendix steering procedure lines 1017-1055.
- Evidence/derivation: A latent direction can induce a behavior when added to the residual stream without being the direction naturally responsible for that behavior. Necessity would require a clearly specified ablation/removal protocol, held-out behavioral metrics, and controls against random directions, matched-norm directions, neighboring latents, and prompt selection. The paper claims validation "via steering, ablation, and max-activation" at line 596, but the appendix gives detailed steering response generation only. I did not find a comparably precise ablation protocol, sample counts for most behavioral metrics, random seeds, confidence intervals, or statistical tests.
- Severity: Major.
- Consequence for acceptance: The strongest causal claims are overstated. The evidence supports "candidate behavior-controlling directions under steering" more than "latents causally responsible for fine-tuned behaviors."
- Confidence: High.

### 6. Evaluation prompt sets and steering scale do not support robust mitigation claims

- Candidate error: The steering/mitigation evaluation uses small hand-crafted prompt sets and extreme steering strengths without uncertainty or a fully specified normalization procedure.
- Exact locations: appendix B lines 1019-1055 lists nine open-ended prompts, steering strengths from -200 to 200, and fixed generation parameters. Appendix D lines 1072-1097 lists nine harmful prompts and four benign prompts for refusal/safety. Line 1047 says decoder vectors are normalized using model-specific normalization factors but does not define those factors.
- Evidence/derivation: The finite prompt sets are too small to justify broad claims of reliable mitigation. Temperature 0.7 also introduces stochasticity, but no number of samples per prompt, seeds, confidence intervals, or variance estimates are reported. Model-specific steering normalization is critical for comparing across models and latents, but the normalization constants and calculation are absent.
- Severity: Major.
- Consequence for acceptance: Behavioral effects should be treated as illustrative until quantified on held-out prompt suites with uncertainty and controls.
- Confidence: High.

### 7. The "false positive" metric is incorrectly defined

- Candidate error: The paper calls method-level failures "false positives", but the definition describes failures to recover any valid latent, which is a false negative or coverage failure.
- Exact locations: lines 811-820. In particular, line 818 defines a method-level false positive as a method failing to recover any latent that supports causal validation, and lines 819-820 report Delta-Crosscoder as `0% method-level false positives` while DSF and BatchTopK have 40-60%.
- Evidence/derivation: A false positive is a selected latent or method output that appears positive but is not actually behaviorally relevant. A method failing to recover any relevant latent is a false negative. The reported numbers in lines 819-820 are better described as organism coverage or recall, not false-positive rate.
- Severity: Major.
- Consequence for acceptance: The robustness discussion overclaims error control. The evidence may show better coverage, but it does not establish a low false-positive rate.
- Confidence: High.

### 8. Table 1 contradicts the text's metric interpretation

- Candidate error: The paper states that Delta-Crosscoder achieves explained variance within a 1-2% absolute range and does not increase feature collapse, but the reported table contains multiple larger drops and several settings where Delta has worse dead-feature rates than baselines.
- Exact locations: main-text summary lines 522-528; appendix metric interpretation lines 943-952; Table at lines 955-1013.
- Evidence/derivation: Recomputing from the table:
  - Qwen Subliminal: Delta explained variance 76.17 versus TopK-400/DSF 80.07, a 3.90 absolute point gap; dead features 64.2% versus TopK-200 8.3% and DSF 9.5%.
  - Gemma Taboo: Delta explained variance 76.17 versus TopK-400 79.29, a 3.12 point gap; dead features 16.4% versus TopK-400 12.0%.
  - LLaMA SDF Cake: Delta explained variance 72.65 versus TopK-400 77.73, a 5.08 point gap; dead features 55.8% versus DSF 51.9%.
  - LLaMA SDF Abortion: Delta explained variance 72.26 versus TopK-400 77.73, a 5.47 point gap; dead features 56.0% versus DSF 53.3%.
  Delta has the best dead-feature rate in only three of the eight tabled organisms. The paper also claims table metrics are across all evaluated organisms, but the paper says there are 10 organisms and Table 1 reports only eight organism rows.
- Severity: Major.
- Consequence for acceptance: The reconstruction/sparsity safety claim is materially unsupported and partly contradicted by the authors' own table.
- Confidence: High.

### 9. Null experiment is too narrow, and one reported relative-norm value is impossible under the paper's definition

- Candidate error: The null experiment is used to claim robustness against false discovery, but it tests only identical model copies. Separately, the no-finetuning-data ablation reports a relative decoder norm value of 52.5, impossible under the relative decoder norm definition.
- Exact locations: null test lines 823-832; relative decoder norm definition lines 293-301; no-finetuning-data ablation lines 1103-1121, especially line 1112.
- Evidence/derivation: Identical-model comparison checks only the zero-difference case. It does not test nuisance differences such as random seeds, unrelated fine-tuning, prompt-distribution shifts, or response-style shifts, so it cannot establish low false discovery under realistic nonzero differences. Also, `R_base_i = ||d_base_i|| / (||d_base_i|| + ||d_ft_i||)` must lie in `[0,1]`; line 1112 says "The most extreme latent attains a value of 52.5", which is inconsistent unless a different metric is being used without being defined.
- Severity: Major for the robustness overclaim; moderate for the likely numeric/notation error.
- Consequence for acceptance: The null result is a useful sanity check but does not validate the claimed robustness. The 52.5 inconsistency suggests metric accounting needs correction before the ablation can be interpreted.
- Confidence: High.

### 10. ADL/grader comparison is not a controlled method comparison

- Candidate error: The paper claims Delta-Crosscoder matches ADL while reducing analysis overhead, but the comparison uses different protocols and a grading setup that is not reported with sufficient controls.
- Exact locations: Figure caption lines 765-767; evaluation setup lines 779-790; results lines 796-801.
- Evidence/derivation: The paper evaluates Delta-Crosscoder by giving a `GPT-5.2` grader top-5 maximally activating examples and one positive/negative steered response. It then compares to the best reported ADL performance per task because the full interactive task suite and fine-grained per-task scores are unavailable. This is not a matched evaluation. The grader sees author-selected artifacts from the target method, grade 2 merely indicates a broad domain/style signal, and there are no repeated grader calls, alternative graders, confidence intervals, or calibration controls.
- Severity: Major.
- Consequence for acceptance: The non-SAE comparison should be framed as suggestive, not as evidence that Delta-Crosscoder matches ADL.
- Confidence: High.

### 11. Hyperparameter reporting is insufficient for several technical claims

- Candidate error: The appendix gives some headline hyperparameters but omits parameters needed to interpret optimization and steering claims.
- Exact locations: training data lines 505-519; training setup lines 416-421; hyperparameter table lines 882-921; steering normalization line 1047.
- Evidence/derivation: Missing or ambiguous items include actual layer indices per model, optimizer betas/weight decay, learning-rate schedule beyond warmup steps, exact data mixture proportions after the 20M contrastive tokens, whether contrastive/unpaired examples use the same delta loss, actual `K_shared` and `K_delta`, BatchTopK scope, random seeds, number of runs, model-specific steering normalization factors, and held-out evaluation split construction.
- Severity: Moderate.
- Consequence for acceptance: This is partly a reproducibility issue, but it also weakens causal and robustness claims because the reported effects may depend on unspecified choices.
- Confidence: High.

### 12. Minor source artifact issue: `[H]` table placement is used without a visible `float` package

- Candidate error: The source uses `\begin{table}[H]` but I did not find `\usepackage{float}` in the local TeX preamble.
- Exact locations: table starts at line 892; package block lines 1-123.
- Evidence/derivation: The `H` float specifier is normally provided by the `float` package. The submitted PDF exists, so this may be tolerated in the build environment or handled indirectly, but the local source as written appears brittle.
- Severity: Minor.
- Consequence for acceptance: No scientific decision impact by itself.
- Confidence: Medium.

## Limitations and Blockers

- I did not find implementation code, raw evaluation data, training logs, latent tensors, selected prompt lists beyond the appendix examples, random seeds, or grader transcripts in the local artifact directory.
- The figure PDFs could not be text-extracted in this environment, so I relied on the TeX figure captions and table values.
- I did not inspect external OpenReview pages, conference outcomes, citation counts, social-media discussion, or future signals. I used the paper source and its bibliography entries only.

## Confidence Level

Overall confidence: High for the internal mathematical and accounting issues, especially the delta-loss residual identity, relative-norm right-tail inconsistency, false-positive metric error, impossible `52.5` relative-norm value, and Table 1 contradictions. Medium-high for the contrastive-pairing confound because the exact implementation is unavailable.

## Decision Impact

The paper contains an interesting direction, and steering-based evidence may indicate useful behavior-controlling directions. However, the correctness record is not strong enough for high confidence in the central claims as written. The main claims should be materially downgraded unless the authors correct the objective definition, define the latent selection statistic consistently, provide a precise Dual-K/BatchTopK implementation, report controlled steering/ablation results with uncertainty, fix the false-positive and metric-accounting errors, and rerun or reframe the ADL comparison. My correctness-only recommendation is a significant negative adjustment relative to a strong-accept case.
