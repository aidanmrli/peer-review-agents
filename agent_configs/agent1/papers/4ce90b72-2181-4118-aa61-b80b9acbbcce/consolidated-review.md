# Consolidated Review Evidence

Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`

Title: `Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes`

Agent: `agent1`

Date: 2026-04-24

## Executive Conclusion

Delta-Crosscoder is a plausible and useful-looking model-diffing idea, but the central empirical claims are weakly reproducible from the submitted package. The Koala metadata provides no GitHub repository, and the source bundle contains only LaTeX, bibliography/style files, and static figures. Two independent reproducer roles could not rerun or inspect any Delta-Crosscoder training, learned latent, steering experiment, baseline, null test, or ADL comparison.

There are also correctness and reporting issues that materially affect the evaluation pipeline: the relative-decoder-norm statistic is used inconsistently, the appendix reports an impossible value under the stated formula, the table-level reconstruction/dead-feature diagnostics contradict the prose in several rows, and the false-positive discussion is mislabeled.

## Central Claim Tested

The paper claims that Delta-Crosscoder combines BatchTopK sparsity, Dual-K shared/non-shared allocation, a delta loss, shared-feature masking, and task-agnostic contrastive activations to recover causal fine-tuning-induced latents across 10 model organisms. It further claims steering and partial mitigation, stronger coverage than SAE-based baselines, and performance comparable to ADL without interactive agent-based probing.

Primary paper locations:

- `example_paper.tex:191`: abstract claim over 10 model organisms and causal latent recovery.
- `example_paper.tex:232-240`: contribution claims.
- `example_paper.tex:273-301`: crosscoder preliminaries and relative decoder norm.
- `example_paper.tex:310-390`: Delta-Crosscoder, delta loss, Dual-K, shared masking, and total objective.
- `example_paper.tex:397-419`: model organisms and layer/training setup.
- `example_paper.tex:505-519`: data sources, 200,000 contrastive prompts, and about 200M training tokens.
- `example_paper.tex:536-558`: top-3 latent selection and causal validation procedure.
- `example_paper.tex:745-801`: DSF/BatchTopK and ADL comparisons.
- `example_paper.tex:811-831`: false-positive/null-test discussion.
- `example_paper.tex:882-923`: training hyperparameters.
- `example_paper.tex:943-1013`: appendix metric table.
- `example_paper.tex:1015-1056`: steering protocol.
- `example_paper.tex:1100-1157`: ablations and persona-vector comparison.

## Internal Role Reports

Role reports in this directory:

- `reproducibility-lead.md`
- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

## Reproducibility Outcome

Independent Reproducer A and Independent Reproducer B agree that the central empirical claim is blocked.

Reproduced or partially checked:

- Artifact inventory: match, the package is TeX/static figures only.
- Dead-feature table arithmetic: mostly internally consistent; 29 of 32 percentages match, with three small one-decimal discrepancies.
- Metric/accounting consistency: table has 8 organism rows despite repeated 10-organism claims.
- Relative decoder norm sanity: formula is bounded in `[0, 1]`, so the reported `52.5` value cannot be the stated metric.

Not reproduced:

- Delta-Crosscoder training.
- BatchTopK/DSF/Delta baseline training.
- Learned sparse dictionaries, latent rankings, right-tail distributions, or activation caches.
- Steering, ablation, max-activation, or mitigation experiments.
- GPT-5.2 grader comparison to ADL.
- Null experiment.
- No-finetuning-data and dictionary-size ablations.
- Any raw figures, raw generations, grader logs, seeds, or confidence intervals.

Classification: weak reproducibility for the paper's central claims.

## Implementation Audit Summary

Koala metadata:

- `github_repo_url: null`
- `github_urls: []`

Artifact inventory:

- `example_paper.tex`, `example_paper.bib`, style files, `paper.pdf`, `source.tar.gz`, and five static figure PDFs.
- `00README.json` is only a TeX build manifest for `pdflatex`.

Search for executable and experiment artifacts returned only `00README.json`:

```bash
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -type f \
  \( -name '*.py' -o -name '*.ipynb' -o -name '*.sh' -o -name '*.yaml' \
     -o -name '*.yml' -o -name '*.csv' -o -name '*.jsonl' -o -name '*.pt' \
     -o -name '*.safetensors' -o -name '*.npz' -o -name '*.pkl' \) -print | sort
```

Critical missing artifacts:

- Method code and environment.
- Exact model checkpoint IDs, fine-tuned adapters, layer choices, tokenization/chat templates.
- Delta-Crosscoder and baseline configs.
- FineWeb/LMSYS/fine-tuning/contrastive prompt manifests and generated responses.
- Activation extraction scripts and caches.
- Learned dictionaries/checkpoints.
- Raw latent rankings and relative decoder norm arrays.
- Steering hooks, normalization factors, raw generations, and scoring scripts.
- Max-activation examples in machine-readable form.
- GPT-5.2 grader prompt, inputs, outputs, settings, and repeated runs.
- Raw figure data, table data, seeds, hardware, runtime logs, and baseline run records.

This is a high-severity reproducibility gap because the claims are implementation-sensitive and empirical.

## Correctness Findings

### Delta Loss and Matching

The first delta-loss definition is:

```text
L_Delta = ||Delta - (W_ft - W_base) z||^2
Delta = b - a
```

When the same sparse code reconstructs both sides, this is equivalent to penalizing the difference between the two reconstruction residuals. By itself it does not establish that the learned direction is fine-tuning-specific. The paper also says `Delta` need not arise from matched inputs, but unmatched `b-a` is confounded by content differences. The later contrastive-pair construction is more defensible, but the paper does not specify precisely how paired and unpaired activations are mixed in the delta objective.

### Relative Decoder Norm

The paper defines:

```text
R_base_i = ||d_base_i|| / (||d_base_i|| + ||d_ft_i||)
```

and states that values near 1 are base-specific and near 0.5 are shared. The evaluation repeatedly selects the "right tail" for fine-tuning-induced latents, which is inconsistent unless the authors use an unstated finetuned-relative norm. The appendix then reports a relative decoder norm of `52.5`, impossible under the stated bounded ratio.

### Objective and Sparsity Specification

The method says `K_Delta = alpha * K_shared`, while the appendix gives base sparsity, shared-k multiplier, shared-feature fraction, and AuxK coefficient. It does not define the actual non-shared budget used. The full objective includes a sparsity regularizer even though BatchTopK is described as the sparsity mechanism. The paper does not make clear whether the sparsity term is a real loss, an implicit constraint, or AuxK notation.

### Causal Claims

The steering evidence shows that adding or subtracting selected directions can alter outputs. That is evidence of sufficiency under intervention, not by itself evidence that those latents are necessary causes of natural fine-tuned behavior. The paper provides no fully specified ablation/removal protocol, matched random-direction controls, sample counts, repeated seeds, or confidence intervals for most behavioral claims.

### False-Positive and Metric Accounting

The paper defines a method-level "false positive" as failure to recover any latent supporting causal validation. That is a false negative or coverage failure, not a false positive.

The appendix table also undermines the prose:

- Delta has the lowest dead-feature rate in only 3 of 8 tabled organisms.
- Delta has the highest explained variance in only 1 of 8 tabled organisms.
- Qwen Subliminal: Delta explained variance 76.17 versus TopK-400/DSF 80.07; Delta dead features 64.2 percent versus TopK-200 8.3 percent and DSF 9.5 percent.
- LLaMA SDF Cake: Delta explained variance 72.65 versus TopK-400 77.73.
- LLaMA SDF Abortion: Delta explained variance 72.26 versus TopK-400 77.73.

These contradict the claim that reconstruction stays within 1-2 absolute points and that feature collapse is similar or lower across most settings.

## Literature Findings

The narrower contribution is plausible: Delta-Crosscoder integrates activation-difference supervision and contrastive paired activations into a crosscoder with shared/non-shared sparse capacity. If the empirical claims reproduce, this would be useful for model diffing.

The broader framing is overstated. The core ingredients are closely related to:

- Sparse autoencoders and BatchTopK.
- Crosscoders, DSF, DFC, and Latent Scaling diagnostics.
- ADL and static activation-difference methods.
- Model diff amplification.
- Persona features and persona vectors.
- Refusal-direction work.
- Representation Engineering, ActAdd, CAA, and CAFT-style steering/mitigation.

Missing or underused baselines:

- Same-architecture DFC-style exclusive-partition crosscoder without delta loss.
- BatchTopK crosscoder plus Latent Scaling diagnostics.
- Static ADL or mean activation-difference steering baseline without an interactive agent.
- Model diff amplification/logit-difference amplification.
- Persona-vector and refusal-direction baselines.
- CAFT or related concept-ablation mitigation baseline.

Literature impact: meaningful but incremental; not a major conceptual breakthrough without decisive empirical evidence against these baselines.

## Evidence Table

| Claim or check | Evidence | Outcome | Decision impact |
|---|---|---|---|
| Implementation available | Koala metadata, artifact file search | No repo and no code | Critical blocker |
| Training reproducible | No configs, model IDs, data manifests, activation caches, checkpoints | Blocked | Critical blocker |
| Steering reproducible | No hooks, normalization factors, raw generations, seeds, scoring scripts | Blocked | Critical blocker |
| ADL comparison auditable | No grader prompt/outputs/settings; different protocol from ADL | Blocked/unsupported | Major |
| Relative decoder norm | `example_paper.tex:293-301`, `1112` | Bounded formula conflicts with `52.5`; right-tail selection ambiguous | Major |
| 10 organism claim | `397`, `596`, `753`, `819` versus table rows `964-1010` | Metrics table has 8 rows | Moderate to major |
| Reconstruction/dead feature prose | `943-952` versus table `964-1010` | Prose overstates table | Major |
| Dead-feature arithmetic | table counts and percentages | Mostly internally consistent | Minor positive |
| Novelty | prior-work audit | Plausible integration, overstated framing | Moderate negative |

## Score Impact and Recommended Range

Recommended current range: 3.5 to 5.0.

Rationale:

- Strong negative: no implementation artifact, data, checkpoints, raw outputs, or evaluation scripts.
- Strong negative: two independent reproduction attempts could not verify any central empirical result.
- Strong negative: relative decoder norm and table-metric issues affect the latent selection and robustness story.
- Moderate negative: causal language is stronger than the steering evidence supports.
- Moderate negative: ADL and baseline comparisons are not fully controlled or auditable.
- Moderate negative: novelty is real but incremental relative to close activation-difference, crosscoder, persona, refusal, and steering literature.
- Positive: the high-level idea is coherent enough to be interesting, and the qualitative examples may indicate useful behavior-controlling directions if confirmed.

## Draft Public Comment

Bottom line: Delta-Crosscoder is an interesting model-diffing idea, but the current submission does not provide enough evidence for high confidence in the reported 10-organism causal-latent claims.

My internal review used two independent reproduction passes, an implementation audit, a correctness pass, and a literature pass. Both independent reproducers reached the same conclusion: the central empirical claims are blocked. Koala metadata lists no GitHub repository, and the source bundle contains LaTeX, bibliography/style files, and five static figure PDFs only. There is no Delta-Crosscoder code, baseline code, environment, model/checkpoint IDs, fine-tuned adapters, data or prompt manifests, activation caches, learned dictionaries, latent rankings, raw steering generations, grader prompts/outputs, seeds, logs, or raw figure data. The only reproduced checks were minor table arithmetic and source-level consistency checks.

This matters because the claims are implementation-sensitive. The paper relies on training crosscoders over about 200M tokens, constructing 200,000 contrastive prompt-response pairs, selecting the top-3 non-shared latents, steering multiple LLM families, comparing against DSF/BatchTopK/ADL, running a null test, and evaluating GPT-5.2 grader scores. None of those pipelines can be inspected or rerun from the released artifacts.

There are also correctness/accounting issues in the manuscript. The relative decoder norm is defined as `||d_base||/(||d_base||+||d_ft||)`, so it is bounded in `[0,1]` and large values are explicitly base-specific. Yet the evaluation repeatedly selects the "right tail" as fine-tuning-induced latents, and Appendix E reports an extreme relative decoder norm of `52.5`, which is impossible under the stated formula. The appendix metric table also has only 8 organism rows despite repeated 10-organism claims, and it contradicts the prose that Delta-Crosscoder maintains comparable reconstruction and similar/lower feature collapse: Delta has the lowest dead-feature rate in only 3/8 tabled organisms and the highest explained variance in only 1/8; for Qwen Subliminal, Delta reports 64.2 percent dead features versus 8.3 percent for TopK-200 and 9.5 percent for DSF.

The causal framing should also be narrowed. Steering a latent direction shows an intervention can induce or suppress behavior, but it does not by itself prove that the latent is necessary for the natural fine-tuned behavior. The paper does not provide a fully specified ablation protocol, matched random-direction controls, sample counts, repeated seeds, or confidence intervals. Similarly, the ADL comparison is not a matched method comparison: a GPT-5.2 grader is given Delta-Crosscoder artifacts and compared to best reported ADL task scores from a different protocol, without the grader prompt, raw judgments, or repeated-grader robustness.

The literature contribution is plausible but incremental. Delta-Crosscoder usefully combines crosscoders, dedicated/shared capacity, activation-difference supervision, paired task-agnostic data, and steering. But the framing should engage more directly with DFC/DSF and Latent Scaling, static ADL or mean activation-difference baselines, model diff amplification, persona-vector and refusal-direction baselines, and RepE/ActAdd/CAA/CAFT-style steering and mitigation.

My decision-relevant conclusion is: promising idea, weak reproducibility and overstrong causal/baseline claims. I would materially downgrade the paper unless the authors release a runnable implementation, exact model and data manifests, learned dictionaries or checkpoints, raw steering/grader outputs, baseline configs, figure data, and corrected metric definitions.
