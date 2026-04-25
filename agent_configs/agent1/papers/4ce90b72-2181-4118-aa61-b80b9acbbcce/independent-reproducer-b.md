# Independent Reproducer B Report

## Paper

- Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`
- Title: Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes
- Assigned role: Independent Reproducer B
- Task scope: independently assess whether the central claims can be reproduced from the paper artifacts, using a route focused on method equations, evaluation protocol, tables/figures, appendix hyperparameters, and clean-room reimplementation sufficiency.
- Independence note: I did not read `independent-reproducer-a.md` before writing this report.

## Claim Attempted

I attempted to validate the paper's central claim that Delta-Crosscoder reliably isolates causal fine-tuning-induced latents across 10 narrow fine-tuning model organisms, outperforming SAE-based crosscoder baselines and matching non-SAE ADL-style diffing while remaining reproducible from the released artifacts.

My route was deliberately not a full code rerun, because the artifact inventory quickly showed no implementation. Instead, I checked whether a clean-room reproduction could be specified from the method equations, data/evaluation protocol, reported tables, appendix hyperparameters, and released artifacts.

## Evidence Examined

- Koala metadata via `get_paper`: status `in_review`, arXiv ID `2603.04426`, no GitHub URLs (`github_urls: []`), and tarball/PDF storage paths.
- Artifact directory: `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/`
- Main source: `artifacts/example_paper.tex`
- Bibliography: `artifacts/example_paper.bib`
- Figures: `artifacts/figures/*.pdf`
- Build manifest: `artifacts/00README.json`

No Koala comments were needed because Koala metadata reports `comment_count: 0`.

## Commands and Checks

Artifact inventory:

```bash
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -maxdepth 4 -type f | sort
tar -tzf papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/source.tar.gz | sort
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -type f \
  \( -name '*.py' -o -name '*.ipynb' -o -name '*.json' -o -name '*.yaml' -o -name '*.yml' \
     -o -name '*.toml' -o -name '*.sh' -o -name '*.csv' -o -name '*.jsonl' -o -name '*.pt' \
     -o -name '*.safetensors' -o -name '*.pkl' -o -name '*.npz' \) -print | sort
```

Observed:

- The tarball contains `example_paper.tex`, `example_paper.bib`, style files, and five figure PDFs.
- The executable/data/config search returned only `artifacts/00README.json`; there are no Python files, notebooks, configs, activation caches, checkpoints, generated prompt-response files, raw grader logs, or metric outputs.
- `00README.json` is only a TeX build manifest specifying `pdflatex`, not a reproduction manifest.

Method/evaluation source checks:

```bash
rg -n "Delta|Crosscoder|BatchTopK|Dual|shared|non-shared|lambda|dataset|prompt|grader|GPT-5.2|seed|score|TopK|DSF|ADL" \
  papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '300,560p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '560,880p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '880,1165p'
```

Table arithmetic check:

```bash
awk 'BEGIN{print "organism/method dead_pct_check"} /& (Delta|TopK|DSF)/ {
  gsub(/\\\\/,""); split($0,a,"&");
  method=a[2]; dict=a[3]+0; dead=a[5]+0; pct=a[6]+0; calc=100*dead/dict;
  printf "%s dict=%d dead=%d reported=%.1f calc=%.2f diff=%.2f\n", method, dict, dead, pct, calc, pct-calc
}' papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex
```

Observed: the dead-feature percentages in Table 1 are arithmetically consistent with the reported dead counts and dictionary sizes, up to rounding.

Organism-count check:

```bash
awk '/multirow\{4\}\{\*\}/ {
  line=$0; sub(/^.*\}\{/,"",line); sub(/\}.*/,"",line); print line
}' papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | nl -ba
```

Observed: the metrics table has 8 organism rows:

1. LLaMA EM (Extreme Sports)
2. LLaMA EM (Risky Finance)
3. LLaMA EM (Bad Medical)
4. Qwen EM (Extreme Sports)
5. Qwen Subliminal
6. Gemma Taboo (Gold)
7. LLaMA SDF (Cake Bake)
8. LLaMA SDF (Abortion)

This conflicts with repeated claims of 10 model organisms and with the setup statement that LLaMA and Qwen are both evaluated across three EM settings, which would imply 6 EM settings plus 2 SDF, 1 Taboo, and 1 Subliminal.

Manual equation sanity check:

```bash
python - <<'PY'
examples = [(0, 1), (1, 1), (10, 0), (3, 7)]
print('rdn_examples')
for nb, nf in examples:
    denom = nb + nf
    r = None if denom == 0 else nb / denom
    print(f'base_norm={nb}, ft_norm={nf}, R={r}')
print('A reported relative decoder norm of 52.5 cannot be produced by Eq. (rdn) unless a different, undefined metric is used.')
print('metric_table_organism_rows=8; stated_organisms=10; missing_rows=2')
PY
```

Observed: the paper's Relative Decoder Norm definition in Eq. (rdn) is bounded in `[0, 1]`, but Appendix E reports an extreme relative decoder norm of `52.5`. That value is impossible under the stated formula unless the authors use a different, undefined statistic.

## Findings

### 1. Central empirical reproduction is blocked by missing artifacts

The paper's strongest claims are empirical: causal latents across 10 model organisms, steering and ablation effects, coverage over SAE baselines, and ADL-comparable grader scores. None of these can be independently rerun from the release.

The artifacts contain only TeX, bibliography, style files, and figure PDFs. There is no:

- Delta-Crosscoder training code.
- Baseline code for BatchTopK, DSF, or ADL comparison.
- Exact base/fine-tuned model checkpoint IDs or fine-tuned weights.
- Activation extraction code or cached activations.
- FineWeb/LMSYS sampling manifest, prompt IDs, random seeds, or generated contrastive prompt-response data.
- Fine-tuning datasets or exact model organism checkpoints.
- Raw relative-decoder-norm distributions or latent rankings.
- Steering implementation code, hook locations, model-specific normalization factors, or raw steered generations.
- Grader prompt, grader settings, raw grader outputs, or deterministic evaluation logs for the `GPT-5.2` grading comparison.

Result: **blocked**, not reproduced.

### 2. The clean-room method specification is underspecified

The method section gives the high-level idea but leaves several implementation-critical details ambiguous.

Specific issues:

- Lines 310-316 define `Delta = b - a` and state that `a` and `b` need not arise from matched inputs. For unrelated prompts, this difference is not identifiable as a fine-tuning-induced representation shift; it also contains arbitrary input-content differences. Lines 348-357 then introduce contrastive pairs that do impose matched prompt/response structure. The paper does not specify how paired and unpaired activations are mixed in the delta objective or how unpaired deltas avoid becoming content-difference signals.
- Lines 277-286 define the standard crosscoder encoder path as `u_base`, `u_ft`, averaged `u`, and `z = BatchTopK(u)`. In the Delta-Crosscoder section, the delta loss is written directly in terms of `z` and decoders, but the exact encoder path for `z_delta` is not specified for paired, unpaired, or asymmetric contrastive examples.
- Lines 364-365 state `K_delta = alpha * K_shared`, while Appendix Table A reports `Base Sparsity k_base = 200`, `Shared k Multiplier = 2.0`, and `AuxK Coefficient = 1/32`. The relationship between these quantities and the method's `alpha` is not defined. A reimplementer cannot infer the actual shared/non-shared BatchTopK budgets.
- Lines 382-390 include a sparsity regularizer term `lambda_s sparsity(z)`, but the paper also says it uses BatchTopK rather than an L1 penalty. Appendix A does not report `lambda_s` or define the regularizer.
- The single "middle layer" description at lines 418-419 is insufficient for reproduction across Gemma, LLaMA, and Qwen models; exact layer indices are not reported.
- Steering says decoder vectors are normalized with model-specific normalization factors (lines 1042-1047), but those factors are not reported.

These are not cosmetic omissions. They affect which latents are selected, whether the delta loss trains the intended signal, and whether the steering magnitudes are comparable.

### 3. Reported evaluation coverage is internally inconsistent

The paper repeatedly claims evaluation across 10 model organisms (abstract line 191, contributions line 239, setup line 397, evaluation line 596, baseline coverage line 751). The metrics table introduced as covering "all organisms and methods" at lines 943-944 contains only 8 organism rows. The missing rows are most plausibly Qwen EM Risky Finance and Qwen EM Bad Medical, because lines 703-704 say LLaMA and Qwen are considered across three EM settings.

This does not by itself falsify the 10-organism claim, because the omitted organisms may appear only in figures, but it prevents table-level reproduction and makes the "across all evaluated organisms" reconstruction/sparsity claim overstated.

### 4. A key ranking statistic is inconsistent with its own definition

The paper's Relative Decoder Norm is defined as

`||d_base_i|| / (||d_base_i|| + ||d_ft_i||)`

at lines 289-301, so it must be in `[0, 1]` when the denominator is nonzero. Appendix E then reports that the most extreme latent has a relative decoder norm of `52.5` (line 1112). This is mathematically impossible under the stated definition. Either the appendix uses an unreported alternative statistic, or the latent-ranking description is wrong.

This directly affects the main evaluation protocol because latents are selected from the right tail of this statistic (lines 536-540).

### 5. The causal validation protocol is mostly qualitative and not independently auditable

The evaluation chooses the top 3 non-shared latents and validates them by steering, base-model steering, and max-activation inspection. The appendix supplies example prompts and coarse generation parameters, but not:

- number of trials per prompt/strength/latent,
- exact prompts for non-refusal organism-specific datasets,
- random seeds,
- decoding backend,
- intervention code,
- residual stream layer/hook names,
- model-specific activation normalization,
- criteria for "supports causal validation",
- raw generations or labels,
- scorer prompts and model settings for the ADL comparison.

The ADL comparison is especially difficult to reproduce. Lines 779-790 define a separate LLM grader and compare against the best reported ADL performance per task, but the paper does not provide the grader prompt, example inputs, raw top-5 activation examples, steered generations, scorer outputs, or confidence intervals.

### 6. Some reported diagnostics support weak sanity checks, not the central claim

I was able to verify:

- The released source and tarball are internally consistent as a TeX artifact.
- The dead-feature percentages in Table 1 are correctly rounded from the reported counts and dictionary sizes.
- The appendix provides some coarse hyperparameters: expansion factor 5, base sparsity 200, shared multiplier 2.0, shared fraction 20%, delta lambda 0.005, Adam, learning rate `1e-4`, 50,000 steps, batch size 4096, bf16, gradient checkpointing.

These checks are too shallow to reproduce the method's main empirical claims. They validate only arithmetic/reporting fragments.

## Reproduction Outcome

- Match / partial match / mismatch / blocked: **blocked**
- What was reproduced: table dead-feature percentage arithmetic; artifact inventory; a small mathematical consistency check showing the stated RDN statistic cannot produce the appendix's `52.5` value; organism-count inconsistency in the metrics table.
- What was not reproduced: Delta-Crosscoder training, latent recovery, latent ranking, steering effects, ablation effects, null experiment, baseline coverage, ADL comparison, and claimed causal mitigation.

## Limitations and Blockers

Primary blockers:

1. No implementation or runnable scripts.
2. No linked GitHub repository in Koala metadata.
3. No checkpoints, activation caches, generated contrastive data, or raw evaluation logs.
4. Insufficient method specification for a faithful clean-room implementation.
5. No deterministic evaluation records for steering, max-activation analysis, null tests, or LLM-grader comparisons.

Secondary limitation:

- I could not extract figure text using `pdftotext` because that utility is not installed in the environment. This does not materially change the conclusion because the source text and artifact inventory already establish the core reproducibility blockers.

## Agreement With Reproducer A

Not assessed in this report. Per instruction, I did not read Independent Reproducer A's report before writing mine. The coordinator can compare outcomes after both reports are complete.

## Confidence

High confidence that the central empirical claims are not reproducible from the provided artifacts. The absence of code, checkpoints, data manifests, activation caches, raw outputs, and grader logs is directly observed.

Medium-high confidence that a clean-room reproduction would be materially underdetermined. The missing encoder/loss/sparsity details and the RDN inconsistency affect the exact algorithm and latent-selection criterion.

## Decision Impact

Major negative impact. The paper may contain a promising idea, and several qualitative examples are plausible, but the claimed contribution is an empirical method for robust model diffing. Under an agent1 reproducibility standard, the acceptance case requires more than narrative examples and static figures. The authors should release a runnable Delta-Crosscoder implementation, exact model/checkpoint identifiers, data/prompt manifests, activation extraction settings, latent rankings, raw steering generations, grader prompts/outputs, baseline configs, and the missing organism-level metric rows. Without those, the reported result should be treated as weakly reproducible at best and should be materially discounted.
