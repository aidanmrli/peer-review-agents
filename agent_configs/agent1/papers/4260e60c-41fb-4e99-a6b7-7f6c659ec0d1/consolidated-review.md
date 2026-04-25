# Consolidated Internal Review

## Paper

- Paper ID: `4260e60c-41fb-4e99-a6b7-7f6c659ec0d1`
- Title: `Demystifying When Pruning Works via Representation Hierarchies`
- Koala status checked: `in_review`
- Agent: `agent1`
- Review focus: reproducibility, implementation audit, correctness, and literature grounding

## Central Claim Being Tested

The paper argues that pruning has a regime-dependent effect on language models: non-generative tasks such as retrieval and multiple-choice classification are often preserved, while free-running generation can collapse. The proposed explanation is a representation hierarchy in which hidden states and logits remain comparatively stable, but the softmax probability distribution amplifies pruning-induced logit perturbations and those deviations compound through autoregressive history.

Under agent1's review protocol, the minimum acceptable reproduction target was not merely to read the paper tables. At least two independent roles needed to recover the central numerical result, plotted metric, or executable behavior from the submitted artifacts, or to identify a clear end-to-end path that would do so with the released code and documented dependencies.

## Sources Checked

Paper/source artifacts:

- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/main.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/introduction.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/related_works.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/method.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/experiments.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/appendix.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/references.bib`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/figures/`

Author repository:

- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations`
- Remote: `https://github.com/CASE-Lab-UMD/Pruning-on-Representations`
- Local commit: `ba1c25bd4d08c27319bf85d5a75b381fa70279d6`

Koala discussion:

- Existing comment `15f36ff9-c892-4d10-9b6a-d66d5c1d5b35`, which is a bibliography audit and does not affect this reproducibility conclusion.

Forbidden sources were not used. No OpenReview reviews, decisions, acceptance status, citation trajectories, social media commentary, or external commentary about this exact paper were used.

## Commands And Checks

Representative commands run across the team:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
git -C papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations rev-parse HEAD
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1 -maxdepth 3 -type f | sort
find papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations -maxdepth 5 -type f \( -name "*.json" -o -name "*.csv" -o -name "*.jsonl" -o -name "*.log" -o -name "*.pt" -o -name "*.pth" -o -name "*.npy" -o -name "*.pkl" \)
rg -n "Drop-8A|Drop-8M|GSM8K|HumanEval|e5-mistral|58\\.9|69\\.3|22\\.3" papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections -g "*.tex"
python --version
python -m py_compile representation-analysis/transition_layerwise_compare.py representation-analysis/compare_generation_metrics.py representation-analysis/compare_mcq_subspace_metrics.py representation-analysis/generation_forward_utils.py transition_metrics_logging.py intra-layer/main.py intra-layer/lib/data.py intra-layer/lib/prune.py
bash -n intra-layer/scripts/prune.sh
```

Environment and execution result:

- Python: `3.12.12`
- `torch`, `transformers`, `datasets`, and `accelerate` were not installed in the current environment.
- Syntax compilation of the main analysis/pruning scripts succeeded.
- Runtime reproduction of model results was not attempted because it requires heavy dependencies, GPU-scale models, and unavailable dropped/pruned artifacts.

## Role Findings

### Reproducibility Lead

The paper's internal tables support the claimed direction. In `artifacts/sections/method.tex:57-112`, E5-Mistral retrieval averages are 58.9 full, 53.4 Drop-8A, and 56.8 Drop-8M; Mistral multiple-choice averages are 69.3 full, 69.8 Drop-8A, and 64.3 Drop-8M; generative averages are 22.3 full, 13.2 Drop-8A, and 0.8 Drop-8M. These numbers are consistent with the paper's story, but they are table values, not reproduced results.

The author repository contains plausible scripts for hidden/logit/probability metrics, but not raw benchmark outputs, dropped or pruned checkpoints, exact drop lists, cosine/KL logs, plotted numeric data, sampled C4 calibration indices, or a complete benchmark reproduction pipeline. The lead classification is weak reproducibility for the central empirical claim.

### Independent Reproducer A

Reproducer A independently checked the paper source, figures, README, and metric scripts. They could statically verify the direction of the table arithmetic and found code paths matching the intended `h`, `z`, and `p` analysis: `transition_metrics_logging.py` logs hidden similarity, logit/head similarity, vocabulary similarity, KL, and variance estimates; `compare_generation_metrics.py` computes per-step hidden/logit/probability cosine and KL metrics.

Outcome: partial static reproduction only. The attempt did not reproduce numeric figures or benchmark results. Runtime entry points were blocked by missing `torch`, and the stronger blocker was the absence of raw logs, benchmark JSON/output files, dropped-model configs, pruned checkpoints, or saved analysis tensors.

### Independent Reproducer B

Reproducer B took the README/repository route and tested whether a reviewer could recover the paper's tables or figures from the released instructions. Syntax checks passed, but reproduction of table/figure metrics was blocked. The repository contains pre-rendered figures in `figs/` and `docs/figs/`, but no metric logs, result directories, checkpoint directories, `.pt` caches, benchmark output JSON, or plotting scripts sufficient to regenerate the paper figures.

Reproducer B also found that the README commands are not copy-paste complete: the examples require `/path/to/dropped_results`, `/path/to/dense_model`, and `/path/to/pruned_model`. Running from the repository root with the README script name fails because the script is in `representation-analysis/`; running from that subdirectory reaches imports but requires the unavailable ML stack and external model artifacts. Their conclusion is that the released repo supports inspection, not independent recovery.

### Implementation Auditor

The implementation audit found code fragments aligned with the paper, but serious artifact and code-path gaps:

- The public artifact does not reproduce the reported benchmark suite. The paper lists GSM8K, HumanEval, MBPP, NarrativeQA, NQ-Open, multiple MCQ tasks, and BEIR retrieval in `appendix.tex:17-18`, but the repository exposes only partial benchmark templates and no raw benchmark outputs.
- The default Qwen inter-layer story is not fully supported by the released inter-layer dropping pipeline. The paper states that Qwen-2.5-7B-Instruct is the main model in `appendix.tex:8-9`, while the inter-layer post-drop saver supports only a limited model set in the inspected code; a root `modeling_qwen.py` exists but is not a self-contained dropped-Qwen module.
- The README dropped-analysis commands may be no-ops for standard Hugging Face Qwen models: `generation_forward_utils.apply_drop_masks` only sets `drop_attn` and `drop_mlp` attributes if the loaded layers already have those attributes, while the README passes `Qwen/Qwen2.5-7B-Instruct`.
- In pruned mode, `transition_layerwise_compare.py` calls pruned attention with `past_key_value=None`, `use_cache=False`, and `cache_position=None` inside an otherwise cached generation loop. This does not faithfully implement the paper's stated "same dense context, replace only current layer" intervention after prefill.
- The representation-analysis scripts use hardcoded toy prompts rather than the paper's claimed multiple prompts and benchmark coverage.
- Dependencies are incomplete for the advertised code paths; root `requirements.txt` omits packages imported by inter-layer and benchmark utilities.

These issues are decision-relevant because the acceptance case depends on measured discrepancies across pruning methods, models, and representation spaces.

### Correctness Specialist

The correctness pass did not find a fatal algebraic error in the local Taylor expansions. The cosine expansion and local KL approximation are plausible under the stated small-perturbation assumptions.

The specialist nevertheless found several correctness weaknesses:

- KL direction is inconsistent between the theorem and repository metrics. The paper defines `p` as original and `q` as compressed, and states `KL(p || q)`, while the README and layerwise implementation report `KL(p_pruned || p_dense)` in the audited path. KL is asymmetric, so this is not a harmless notation issue for large deviations.
- The "softmax amplifies deviation" language is stronger than the math establishes. The theorem shows local sensitivity governed by weighted logit variance and temperature, not a universal amplification relative to logit-space angular deviation.
- The multiple-choice subspace explanation does not match full benchmark scoring. The released script uses a single toy prompt, lowercase one-token labels `[" a", " b", " c", " d"]`, and renormalized final-token probabilities, while the paper says MCQ benchmarks are evaluated via log-likelihood over candidate options.
- The generation-collapse causal story is not isolated from trajectory and sampling effects. Shared-context layerwise analysis avoids history confounds, but the later free-running generation analysis mixes direct pruning errors with sampled trajectory divergence.
- Seed/statistical treatment is weak for broad claims: visible scripts use single seeds/prompts, while figures report mean/min-max over prompts/steps rather than uncertainty over calibration seeds or evaluation repeats.

The correctness recommendation is a substantial downgrade from the paper's confident mechanistic framing, even though the broad idea remains plausible.

### Literature Specialist

The literature pass did not find a novelty-killing prior work. Wanda, SparseGPT, ShortGPT, Gromov et al., and Layer Drop/LLM-Drop already establish redundancy, pruning, and representation/layer-similarity premises. The submitted paper's most distinct contribution is the explanatory synthesis across hidden, logit, and probability spaces, including softmax/KL variance approximations, autoregressive temporal compounding, and categorical-token subspace behavior.

The framing is accurate only if scoped to training-free or post-hoc pruning/dropping. It is overstated if presented as a general claim about "network pruning" in generative settings. LLM-Pruner-style recovered structural pruning and LayerSkip-style trained early-exit/layer-skipping systems are relevant counterpoints that should be explicitly distinguished. The literature weakness is moderate, not fatal.

## Reproduction Outcome

The central empirical claim was not reproduced by two independent internal roles.

- Reproducer A: partial static corroboration only; no numeric result or figure recovery.
- Reproducer B: reproduction blocked from README/repo route; no table/figure recovery.
- Implementation audit: repository contains useful fragments but not a reliable reproduction package.

Classification: weak reproducibility for the paper's core empirical and mechanistic claims.

## Acceptance Consequence

The paper has a plausible and potentially useful explanation, but the evidence package is not strong enough for high confidence. The public artifact does not let an independent reviewer recover the reported benchmark table, plotted representation metrics, or exact Qwen/Mistral pruning setups. Several implementation and correctness issues further weaken the claimed mechanism.

This should materially reduce the score under a reproducibility-first ICML review. The appropriate public comment should not claim the paper is refuted; it should state that the headline claim is not independently reproducible from the released artifacts, and that the mechanistic interpretation is currently supported by paper figures and code skeletons rather than recoverable evidence.

## Public Comment Basis

The public Koala comment should focus on:

1. No two internal roles reproduced the central numeric claim.
2. Missing raw outputs/checkpoints/configs/drop lists/logs/figure data make the evidence package weak.
3. README dropped-mode commands are underspecified and may not apply actual drops for standard Qwen layers.
4. The pruned-layer attention hook does not preserve cached context, weakening the layerwise mechanism check.
5. The MCQ subspace and KL-direction issues make the explanatory framing less certain.

Score impact if converted into a verdict later: weak-reject range unless authors provide exact artifacts and clarify the implementation/correctness concerns.
