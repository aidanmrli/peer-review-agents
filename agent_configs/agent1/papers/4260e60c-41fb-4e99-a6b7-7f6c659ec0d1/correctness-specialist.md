# Correctness Specialist Report

## Paper

- Paper ID: `4260e60c-41fb-4e99-a6b7-7f6c659ec0d1`
- Title: `Demystifying When Pruning Works via Representation Hierarchies`
- Role: Correctness Specialist
- Author repository inspected: `repos/Pruning-on-Representations`, commit `ba1c25bd4d08c27319bf85d5a75b381fa70279d6`

## Claim Tested

The central correctness claim is that pruning leaves embedding/logit representations relatively stable, but softmax maps small logit perturbations into larger probability-space deviations; these deviations then persist across autoregressive steps, explaining why pruning works better for non-generative tasks than for generation.

## Sources and Commands

Sources inspected:

- `artifacts/main.tex`
- `artifacts/sections/method.tex`
- `artifacts/sections/experiments.tex`
- `artifacts/sections/appendix.tex`
- `repos/Pruning-on-Representations/README.md`
- `repos/Pruning-on-Representations/transition_metrics_logging.py`
- `repos/Pruning-on-Representations/representation-analysis/generation_forward_utils.py`
- `repos/Pruning-on-Representations/representation-analysis/transition_layerwise_compare.py`
- `repos/Pruning-on-Representations/representation-analysis/compare_generation_metrics.py`
- `repos/Pruning-on-Representations/representation-analysis/compare_mcq_subspace_metrics.py`
- `repos/Pruning-on-Representations/intra-layer/main.py`
- `repos/Pruning-on-Representations/intra-layer/lib/data.py`
- `repos/Pruning-on-Representations/inter-layer/scripts/benchmark/benchmark_lm_eval.sh`

Commands run:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,220p' skills/correctness-specialist.md
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1 -maxdepth 3 -type f | sort
rg -n "Taylor|KL|softmax|amplif|causal|subspace|seed|generative|non-generative|candidate|temperature|hierarch|prun" ...
nl -ba artifacts/sections/method.tex
nl -ba artifacts/sections/experiments.tex
nl -ba artifacts/sections/appendix.tex
nl -ba repos/Pruning-on-Representations/transition_metrics_logging.py
nl -ba repos/Pruning-on-Representations/representation-analysis/compare_generation_metrics.py
nl -ba repos/Pruning-on-Representations/representation-analysis/compare_mcq_subspace_metrics.py
nl -ba repos/Pruning-on-Representations/representation-analysis/generation_forward_utils.py
git -C repos/Pruning-on-Representations rev-parse HEAD
python - <<'PY'
import math
p=[0.7,0.2,0.1]; q=[0.35,0.45,0.20]
def kl(a,b): return sum(x*math.log(x/y) for x,y in zip(a,b))
print(round(kl(p,q), 6), round(kl(q,p), 6))
PY
```

`pdftotext` was not installed, so I used the submitted LaTeX sources and repository code as the exact paper/source artifacts.

## Non-fatal Math Checks

I did not find a fatal algebraic error in the local Taylor expansions themselves. The cosine expansion in Appendix C.1/C.2 is the standard second-order angular-deviation expansion, and the KL derivation in Appendix B is correct for the paper's stated convention: `p = softmax(z/T)` is the original distribution, `q = softmax((z + Delta z)/T)` is the compressed distribution, and `KL(p || q) ~= Var_p(Delta z)/(2T^2)` (`appendix.tex:29-33`, `appendix.tex:124-153`, `appendix.tex:215-223`). Theorem 2's reduction to `Var_r(Delta z)/(2T^2)` with `r_i = p_i^2 / ||p||^2` is also algebraically plausible under the local first-order softmax perturbation (`appendix.tex:460-481`, `appendix.tex:550-684`).

The correctness problems are therefore not a simple symbolic sign error in the appendix derivation. They are mismatches between theorem statements, plotted/measured quantities, task framing, and the causal/statistical interpretation of the evidence.

## Findings

### 1. KL direction is inconsistent between the theorem and repository metrics

- Candidate error: the paper derives `KL(p_original || q_compressed)`, while the repository's layerwise `REAL_KL` path and README document `KL(p_pruned || p_dense)`.
- Exact locations:
  - Paper definition: `appendix.tex:29-33` defines `p` as original and `q` as after compression.
  - Paper theorem: `experiments.tex:203-212` and `appendix.tex:215-223` state `KL(p || q) ~= Var_{i~p}(Delta z_i)/(2T^2)`.
  - README metric contract: `README.md:129-132` says probability metrics report `KL(p_pruned || p_dense)`.
  - Layerwise code: `transition_metrics_logging.py:100-105` sets `q = softmax(residual_head/T)`, `p = softmax(output_head/T)`, then computes `F.kl_div(log_q, p)`, i.e. `KL(p || q)` in PyTorch semantics. Under `transition_layerwise_compare.py:83-95`, `residual` is the dense output and `hidden_states` is the pruned output, so this is `KL(pruned || dense)`.
  - A second script uses the opposite convention: `compare_generation_metrics.py:82-86` computes `KL(probs_a || probs_b)`, and its call sites pass dense first and target second (`compare_generation_metrics.py:261-300`).
- Evidence/derivation: KL is direction-sensitive. A toy check gives `KL(p||q)=0.253702` and `KL(q||p)=0.260947` for `p=[0.7,0.2,0.1]`, `q=[0.35,0.45,0.20]`; the difference can grow for sharper distributions and larger pruning shifts.
- Severity: major.
- Consequence for acceptance: the theorem-to-figure validation is not cleanly specified. The second-order approximation may make the two directions close locally, but the paper uses these plots to explain large generation collapse, precisely where direction and base weighting are no longer a harmless convention. This weakens the claimed empirical validation of Theorem 3.

### 2. The "softmax amplifies deviation" claim is stronger than what the math establishes

- Candidate error: the paper frames softmax as an amplification mechanism, but the theorem establishes local sensitivity to `Var_r(Delta z)/T^2`, not a universal amplification relative to logit-space deviation.
- Exact locations:
  - Main claim: `main.tex:66-69` and `method.tex:7-10` say softmax amplifies pruning-induced deviations.
  - Theoretical statement: `experiments.tex:177-201` states that probability-space cosine deviation is approximated by `Var_r(Delta z)/(2T^2)`.
  - Derivation: `appendix.tex:460-481` and `appendix.tex:675-684` derive a local first-order/second-order approximation.
  - Logit-space comparator: `experiments.tex:149-174` and `appendix.tex:379-454` use a different angular metric governed by orthogonal magnitude relative to `z`.
- Why unsupported: the theorem does not prove that `1 - CosineSim(softmax(z), softmax(z + Delta z))` exceeds the logit-space angular deviation. It says the probability-space angular deviation is locally controlled by a weighted logit variance and temperature. For example, uniform logit shifts have zero softmax effect, and high-temperature softmax can damp changes. The paper has empirical evidence for selected traces, but the theorem itself supports "softmax can be sensitive under high weighted variance/low T," not the broader causal phrase "softmax amplifies deviations."
- Severity: moderate.
- Consequence for acceptance: the central mechanism is plausible but overclaimed. This should reduce confidence in the explanatory generality, though it is not by itself a fatal mathematical flaw.

### 3. Candidate-subspace analysis does not match the stated non-generative benchmark evaluation

- Candidate error: the paper explains multiple-choice robustness using a four-token A/B/C/D probability subspace, but the released script tests one toy prompt with lowercase one-token labels and renormalized final-token probabilities. This is not equivalent to the paper's stated benchmark evaluation via candidate-option log-likelihood.
- Exact locations:
  - Paper simplification: `method.tex:131-136` models multiple-choice as `argmax_{j in C} p(j|x)` over A/B/C/D-like tokens.
  - Benchmark description: `appendix.tex:18` says non-generative tasks are evaluated via log-likelihood over candidate options.
  - Subspace claim: `experiments.tex:290-297` claims categorical-token subspace stability explains multiple-choice robustness.
  - Representative appendix prompts use uppercase `A) ... B) ... C) ... D) ...`: `appendix.tex:821-835`.
  - Released subspace script uses one prompt with lowercase `a) ... d)` and hardcoded `option_tokens = [" a", " b", " c", " d"]`: `compare_mcq_subspace_metrics.py:118-125`, `compare_mcq_subspace_metrics.py:198-205`.
  - It then extracts only the final-step token IDs, renormalizes them within the four-token subspace, and computes cosine/KL there: `compare_mcq_subspace_metrics.py:218-257`.
- Why technically wrong or unsupported: many stated benchmarks score full candidate continuations or answer strings, not a single final lowercase label token. Even for label-only prompts, tokenization is case-sensitive and prompt-format-sensitive; `" a"` is not guaranteed to represent the same answer token as `"A)"`, `" A"`, or the candidate text. Renormalizing four chosen token probabilities also changes the quantity from full-option log-likelihood to conditional probability inside an analyst-selected subspace.
- Severity: major.
- Consequence for acceptance: the subspace explanation for non-generative robustness is under-supported and may not apply to the benchmark numbers in Table 1/Figure 7. This is a decision-relevant correctness gap because it targets one of the paper's main explanatory claims.

### 4. The generative-collapse causal attribution is not isolated from trajectory and sampling effects

- Candidate error: the paper attributes later-step divergence to pruning-induced probability-space shifts propagating through autoregressive history, but the experiments do not isolate this causal path from separate sampled trajectories, seed dependence, or layer-removal effects.
- Exact locations:
  - The paper correctly says the layerwise analysis uses a shared dense-model context to avoid history confounds: `experiments.tex:73-80`.
  - The later causal claim says generated-token differences lead to sharp increases and generation collapse: `experiments.tex:255-262`.
  - The appendix introduces informal functions `F(Delta W, x_t)` and `F(Delta x_{0:t})` rather than a quantitative bound or measured decomposition: `appendix.tex:724-811`.
  - The generation utility samples when `temperature != 0.0`: `generation_forward_utils.py:117-123`.
  - The final-step comparison script hardcodes one prompt and resets `torch.manual_seed(42)` before dense and target generation: `compare_generation_metrics.py:197-205`, `compare_generation_metrics.py:246-257`, `compare_generation_metrics.py:275-286`.
- Why unsupported: same-seed stochastic generation from two different distributions can diverge because a small early probability change crosses a sampling threshold, after which the models condition on different text. That is a real deployment failure mode, but it is not the same as proving that softmax amplification, rather than sampling path bifurcation or pruned-model quality loss, is the dominant causal mechanism. The paper would need fixed-token-trajectory counterfactuals, multi-seed generation, or a measured decomposition of direct pruning errors versus history-induced errors.
- Severity: major.
- Consequence for acceptance: the paper's causal explanation should be treated as a plausible hypothesis supported by illustrative evidence, not as established mechanism. This materially lowers confidence in the mechanistic contribution.

### 5. Seed and statistical validity are weak for broad empirical conclusions

- Candidate error: broad claims are made from benchmark tables and representative analysis figures without enough visible seed/statistical treatment in the paper or released scripts.
- Exact locations:
  - Figure 2 reports mean curves and min-max shaded ranges over prompts/steps, not confidence intervals: `experiments.tex:97-104`.
  - Appendix says intra/inter-layer pruning masks use 128 randomly sampled C4 sequences: `appendix.tex:11-15`.
  - Intra-layer pruning exposes a single default seed and sample count: `intra-layer/main.py:38-39`, sets NumPy/Torch seeds once at `intra-layer/main.py:53-55`, and C4 sampling uses Python `random.seed(seed)` at `intra-layer/lib/data.py:59-77`.
  - Analysis scripts hardcode one prompt and one generation seed: `transition_layerwise_compare.py:244-246`, `compare_generation_metrics.py:197-205`, `compare_mcq_subspace_metrics.py:118-125` and `compare_mcq_subspace_metrics.py:139-145`.
  - The benchmark shell lists tasks/few-shot settings but no repeated seeds or variance reporting: `benchmark_lm_eval.sh:11-39`.
- Why unsupported: pruning masks, sampled decoding, and benchmark few-shot evaluation can vary materially with calibration sample, prompt formatting, and random seed. Min-max bands over prompts/steps are not statistical uncertainty. The paper's cross-model/cross-task claims may still be true, but the submitted evidence does not quantify robustness to these sources of variation.
- Severity: moderate to major.
- Consequence for acceptance: this does not refute the reported numbers, but it makes the empirical support materially weaker than the confident explanatory framing suggests.

## Final Synthesis and Score Impact

The local Taylor/KL algebra is mostly sound under its stated small-perturbation assumptions, but the paper's correctness is weakened by mismatched KL conventions in the artifacts, over-strong softmax-amplification language, a candidate-subspace analysis that does not match the stated multiple-choice evaluation, insufficient causal isolation for generation collapse, and weak seed/statistical treatment. These are decision-relevant issues because they affect the paper's main explanatory mechanism, not merely presentation.

My correctness recommendation is a substantial downgrade from the paper's current framing. I would not call the work mathematically invalid, but I would mark the mechanistic and non-generative-subspace claims as only partially supported unless the authors clarify KL direction, reproduce subspace analysis using actual benchmark option likelihoods, and add multi-seed/counterfactual evidence for the autoregressive causal story.
