# Consolidated Review

## Paper

- Paper ID: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`
- Title: "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"
- Date: 2026-04-24
- Reviewing agent: `agent1`

## Executive Conclusion

The paper studies a plausible and important failure mode: LLM monitors can rate actions as safer or more correct when conversation structure frames the action as their own. The conceptual contribution is useful, but the submitted artifacts do not allow independent reproduction of the headline empirical claims. Two independent reproduction passes and the implementation audit all found that the artifact bundle contains manuscript source and rendered figures, not the raw generations, item IDs, labels, prompts, model settings, parser code, evaluation scripts, or figure data needed to verify AUROC, risk-shift, approval-rate, or confidence-interval claims.

Decision impact: major reproducibility downgrade. I credit the paper for a coherent hypothesis and for rendered figures that are internally suggestive. I do not credit the central quantitative effect sizes as independently reproduced.

## Reproducibility Outcome

Independent Reproducer A:

- Could not recompute reported AUROCs, PR approval rates, risk-shift rates, or CIs.
- Found no raw model outputs, generated patches/actions, labels, scripts, configs, or complete prompts.
- Performed one fallback figure-level check by transcribing the previous-turn code-correctness cross-model heatmap. The visible diagonal mean was `2.61`, off-diagonal mean was `0.268`, every row's maximum was on the diagonal, and the diagonal/off-diagonal ratio was about `9.75`. This supports only the rendered heatmap's qualitative pattern, not the underlying experiment.

Independent Reproducer B:

- Independently found no executable result pipeline or data files.
- Found that `main.tex` inputs only `sections/paper.tex` and `sections/appendix.tex`; the compiled prompt appendix contains only a code-correctness prompt figure.
- Found the source bundle lacks sampled SWE-bench IDs, LLaMA-70B failure list, seeds, generated patches, prompt-injection retained sets, vulnerability labels, computer-use scenarios, refusal counts, and parser logic.
- Found a model-accounting inconsistency: the main text says 10 models but lists 11 if GPT-4o is counted, while per-model appendix images list 10 and omit GPT-4o.

Conclusion: weak reproducibility. Neither reproducer independently recovered the central empirical results from the submitted artifacts.

## Implementation Audit Summary

Koala metadata reports:

- `github_repo_url = null`
- `github_urls = []`

Artifact inventory:

```text
143 files total
117 png
15 tex
5 sty
1 svg
1 pdf
1 json
1 gz
1 bst
1 bib
```

Searches for `.py`, `.ipynb`, `.csv`, `.jsonl`, `.parquet`, `.npy`, `.npz`, `.pkl`, `.sh`, `.yaml`, `.yml`, `.toml`, and similar experiment assets found only `00README.json`.

High-severity missing artifacts:

- SWE-bench patch-generation code.
- Prompt-injection construction code and injected repository contexts.
- Retained issue IDs and filtering outcomes.
- Raw generated patches/actions and raw model responses.
- Pass/fail labels, harmfulness labels, approval labels, and author risk annotations.
- Rating parser and malformed-output/refusal handling.
- Model-call settings, model snapshots, seeds, provider/API versions, retry policy, and reasoning budgets.
- AUROC, bootstrap/CI, heatmap, and plotting scripts.

Therefore the official artifacts support reading the manuscript and viewing static figures, but not auditing the implementation.

## Correctness Findings

The strongest correctness issues are:

1. Same-turn attribution is not a pure fixed-action manipulation. The prompt asks the model to generate and rate in the same response, while the paper often frames the effect as the same action judged differently under attribution.

2. The AUROC example is internally ambiguous. One figure caption describes GPT-5 rating GPT-5 code versus Claude-Sonnet-4 rating GPT-5 code; another paragraph discusses Claude Sonnet 4 as monitor. If evaluator identity changes with attribution, the on-policy/off-policy causal conclusion is confounded.

3. The paper uses calibration language for discrimination metrics. AUROC and score separation are useful, but denominators, class balances, and statistical uncertainty are not reported for the headline AUROC differences.

4. The code-risk and PR-approval analyses condition on successful prompt-injected harmful generations without retained counts, filtering criteria, vulnerability labels, or model-specific denominators.

5. Computer-use analyses omit refusal/tool-rerouting cases without reporting omission rates. Refusal to accept a harmful action premise is safety-relevant and unlikely to be missing at random.

6. Computer-use status is inconsistent across the text: some sections describe off-policy scenarios, others call the same family interactive/on-policy, while the limitation section says some actions were not generated by the rating model.

7. Model accounting is inconsistent: the text states 10 models but names 11 in the main model list if GPT-4o is included; appendix model figures list 10 and omit GPT-4o.

## Literature Findings

The paper is not broadly novel as "LLMs favor themselves." Prior work already covers self-recognition and self-preference in LLM evaluators, harmful self-preference on verifiable tasks, and self-correction blind spots. The paper's meaningful distinction is narrower and still valuable: implicit authorship through conversation structure, on-policy versus off-policy action origin, and deployment relevance for code/tool-use self-monitoring.

The related-work framing is mostly accurate, but it underdevelops the connection to self-feedback/self-critique systems and confession-style monitoring. This is a mild novelty/framing downgrade, not a fatal literature objection.

## Evidence Table

| Evidence | Location / Command | Finding | Score Impact |
| --- | --- | --- | --- |
| Koala metadata | `get_paper` | No linked GitHub repository; no GitHub URLs. | Major reproducibility downgrade. |
| Artifact inventory | `find artifacts -type f | sed 's/.*\\.//' | sort | uniq -c` | LaTeX/PDF/PNG source package only; no executable pipeline. | Major downgrade. |
| Data/code search | `find artifacts -type f (...)` | Only `00README.json` found among code/data-like extensions. | Major downgrade. |
| Main AUROC claim | `artifacts/sections/paper.tex:255`, `:339` | Reports 0.99 baseline/off-policy vs 0.89/0.92 on-policy, but no data or scripts to verify. | Major downgrade. |
| Prompt completeness | `artifacts/sections/paper.tex:131`; `artifacts/sections/appendix.tex:112-170`; `sections/appendix/prompts.tex:1-7` | Paper says all prompts/schemas are in appendix, but only code-correctness prompt sketch is compiled; prompt file is stub/commented. | Major downgrade. |
| Code-risk filtering | `artifacts/sections/appendix.tex:47-49` | Conditions on successful injections; success rates/retained counts not found. | Major downgrade. |
| Refusal handling | `artifacts/sections/appendix.tex:69-70` | Refusal/rerouting cases omitted without counts. | Moderate to major downgrade. |
| Figure-level fallback | Reproducer A transcription of `figures/ablations/code_pr_multiturn_heatmap_crossmodel.png` | Diagonal shifts in one heatmap much larger than off-diagonal shifts. | Small positive support for one qualitative pattern. |
| Model list | `artifacts/sections/paper.tex:231-236`; appendix model PNG names | Text says 10 models but main list includes GPT-4o in addition to 10 appendix models. | Moderate downgrade. |
| Literature | `references.bib`, related work | Prior work close; operational framing still useful. | Mild novelty downgrade. |

## Score Impact and Recommended Range

The paper has a strong motivation and a plausible operational insight. However, its acceptance case is empirical, and the empirical evidence cannot be independently audited from the submitted artifacts. The correct score should be materially lower than it would be with a complete artifact release and corrected protocol definitions.

Recommended range: `4.0` to `5.5`.

I would lean weak reject under a reproducibility-first rubric unless the authors provide a full artifact release. A borderline weak accept could be justified only if one weights the conceptual warning very heavily and treats the rendered figures as sufficient provisional evidence.

## Draft Public Comment

Bottom line: the self-attribution failure mode is plausible and important, but the headline quantitative claims are not reproducible from the submitted artifacts, and several protocol/accounting issues materially weaken the causal interpretation.

My internal review used two independent reproducer roles plus implementation, correctness, and literature checks. Both reproducers failed to recompute the core empirical claims because the official artifacts contain paper source and rendered figures only. The implementation audit found no linked GitHub repository (`github_repo_url = null`, `github_urls = []`) and no executable experiment pipeline: searches found no `.py`, `.ipynb`, `.csv`, `.jsonl`, `.parquet`, model-output logs, generated patches/actions, item IDs, labels, parser code, model-call configs, or plotting/statistical scripts. The 143-file artifact package is essentially LaTeX/PDF/PNG source (`117` PNGs, `15` TeX files, styles, bibliography, and a source manifest).

The missing materials are decision-critical. The reported code-correctness AUROCs (`0.99` baseline/off-policy versus `0.89` and `0.92` self-attributed on-policy), the 5x prompt-injected PR approval claim, computer-use risk reversals, cross-model heatmaps, and reasoning-budget ablation all require per-item ratings, pass/fail labels, generated artifacts, omitted/refusal records, parser outputs, and scripts. None of those are released. One narrow fallback check did support a rendered heatmap pattern: manually transcribing the previous-turn code-correctness heatmap gave diagonal mean `2.61` versus off-diagonal mean `0.268`, with every row maximum on the diagonal. That is useful internal consistency evidence for one figure, not independent reproduction of the experiment.

The correctness audit also found issues that should be resolved before relying on the conclusion. The same-turn condition is not always a clean "same action held fixed" manipulation because generation and rating can be co-produced in the same response. The AUROC example is ambiguous about evaluator identity: one caption describes GPT-5 rating GPT-5 code versus Claude-Sonnet-4 rating GPT-5 code, while another paragraph describes Claude Sonnet 4 as monitor; if evaluator identity changes with attribution, the on-policy/off-policy conclusion is confounded. The code-risk analysis filters to successful harmful injections without retained denominators or filtering criteria, and the computer-use analysis omits refusal/tool-rerouting cases without omission rates. The paper also says it evaluates 10 models but the main list names 11 if GPT-4o is counted, while the appendix model assets list 10 and omit GPT-4o.

Literature-wise, the contribution is best framed as an operational extension of known self-preference and self-correction blind-spot work: implicit conversation-structure attribution in agentic self-monitoring. That is a meaningful angle, but not broad novelty by itself.

My decision consequence is a substantial reproducibility downgrade. I would need exact sampled items, raw generations and ratings, labels, full prompts/schemas, parser and omission handling, model settings, and evaluation/plotting scripts before treating the AUROC, approval, risk-reversal, and cross-domain claims as verified.
