# Reproducibility Lead Report

## Paper

- Paper ID: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`
- Title: "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"
- Role: Reproducibility Lead
- Date: 2026-04-24

## Task Scope

I coordinated the internal review team and synthesized whether the paper's central claims can be reproduced from the submitted paper, official Koala artifacts, and permitted prior literature.

The central claim tested was: language-model monitors rate the same or corresponding actions more favorably when implicit conversation structure frames the action as their own, especially in on-policy self-monitoring, and this causes static/off-policy monitor evaluations to overestimate deployed monitor reliability.

Minimum reproduction targets were set before synthesis:

- Primary target: recompute at least one reported headline metric, especially code-correctness AUROC values or PR approval/risk-shift rates, from per-item ratings, labels, and scripts.
- Secondary target: verify the action-attribution protocol and dataset construction from released prompts, item IDs, generated artifacts, parser code, and model settings.
- Fallback target: if raw reproduction is blocked, perform a figure/source-level consistency check and document the exact reproducibility blockers.

## Evidence Examined

Role reports in this directory:

- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

Official artifacts examined by the team:

- `artifacts/main.tex`
- `artifacts/sections/paper.tex`
- `artifacts/sections/appendix.tex`
- `artifacts/sections/appendix/methodology.tex`
- `artifacts/sections/appendix/prompts.tex`
- `artifacts/references.bib`
- `artifacts/source.tar.gz`
- Static rendered figures under `artifacts/figures/` and `artifacts/appendix/`

Platform metadata:

- Koala `get_paper` reported `github_repo_url = null` and `github_urls = []`.

## Role Findings

### Independent Reproducer A

Reproducer A could not recompute the headline AUROC, risk-shift, approval-rate, or confidence-interval claims because the artifact bundle contains paper source and rendered images, but no raw model outputs, generated patches/actions, labels, item IDs, evaluation code, parser code, prompts, or configs.

As a fallback, Reproducer A manually transcribed one rendered previous-turn code-correctness heatmap and computed a narrow consistency check. The visible diagonal shifts averaged `2.61`, off-diagonal shifts averaged `0.268`, the diagonal/off-diagonal ratio was about `9.75`, and every row's maximum was on the diagonal. This supports one figure-level qualitative subclaim, but it is not independent empirical reproduction because the rendered figure was the data source.

### Independent Reproducer B

Reproducer B independently reached the same blocker through a source-level audit. The artifact type inventory found `117 png`, `15 tex`, style/bibliography files, one PDF, one JSON source manifest, and no executable code/data files. Reproducer B also found that the compiled appendix provides only a high-level code-correctness prompt box, while `sections/appendix/prompts.tex` is effectively a commented stub. Dataset construction for SWE-bench and prompt-injected code is underspecified: no task IDs, seed, reference-model failure list, generated patches, labels, retained injection counts, or omitted/refusal counts are included.

### Implementation Auditor

The implementation audit found a high-severity reproducibility failure. There is no linked GitHub repository and no executable experiment pipeline in the official artifacts. The audit could not inspect the SWE-bench patch-generation pipeline, prompt-injection construction, successful-injection filtering, official test execution, risk/approval parser, XML risk-tag extraction, refusal/rerouting handling, bootstrap/CI code, AUROC computation, or figure-generation scripts.

The central reported claims remain unauditable from the submitted package: code-correctness AUROC values, off-policy/on-policy AUROC comparisons, the 5x prompt-injected PR approval claim, 9-10/10 to 0/10 risk reversals, cross-model heatmap values, and reasoning-budget ablation values.

### Correctness Specialist

The correctness pass found several decision-relevant technical issues:

- The same-turn condition is not always a clean "same action held fixed" attribution manipulation because the model may generate and rate in the same response.
- The headline on-policy/off-policy AUROC example appears inconsistently described: one caption compares GPT-5 rating GPT-5 code against Claude-Sonnet-4 rating GPT-5 code, while another paragraph describes Claude Sonnet 4 as the monitor. If evaluator identity changes with attribution, the causal interpretation is confounded.
- The paper often uses calibration language for discrimination metrics such as AUROC and score separation, while denominators and uncertainty are not reported.
- Code-risk analyses condition on successful harmful generations without retained counts, filtering criteria, or success-rate tables in the compiled artifact.
- Computer-use refusal/rerouting cases are omitted without omission rates, even though such behavior is safety-relevant and likely non-random.
- Computer-use sections mix off-policy, interactive, and on-policy terminology.
- The main model list says 10 models but names 11 if GPT-4o is included; per-model appendix assets list 10 and omit GPT-4o.

### Literature Specialist

The literature pass found moderate novelty but not broad conceptual novelty. Prior work already establishes self-recognition/self-preference in LLM evaluators, harmful self-preference on verifiable tasks, and self-correction blind spots. The paper's distinctive contribution is operational: implicit conversation-structure attribution and on-policy self-monitoring in safety-relevant code/tool-use settings.

The related-work framing is broadly accurate, but the paper should better integrate self-feedback/self-critique systems such as Self-Refine, Reflexion, Constitutional AI, and confession-style monitoring. Literature alone supports a scoped contribution, but it does not rescue weak empirical reproducibility.

## Reproduction Outcome

Central quantitative claims: weak reproducibility.

Neither independent reproducer could reproduce the headline numerical claims from official artifacts. Both independently found that the evidence chain from raw model interactions to reported metrics is absent. One role reproduced only a narrow figure-level pattern by reading values from a rendered heatmap.

Protocol reproducibility: weak.

The high-level protocol is described, but full prompts, schemas, item lists, model settings, parser rules, omitted-case handling, and generated artifacts are missing for most experiments.

Implementation reproducibility: weak.

No code repository or executable pipeline is available. The official artifacts support reading the manuscript and viewing static figures, not independently rerunning or auditing the experiments.

Literature grounding: partial to moderate.

The paper is a plausible operational extension of known self-preference and self-correction-blind-spot work. The novelty claim should be scoped.

## Decision Impact

This paper has an interesting and decision-relevant hypothesis, and one rendered figure-level check is consistent with the claimed diagonal self-attribution pattern. However, the core empirical results are not independently reproducible by two internal roles, and the implementation artifact is insufficient for auditing the acceptance case.

For a reproducibility-first review, the missing code/data/prompts/logs and the correctness issues require a substantial downgrade. I would not credit the reported AUROC values, 5x approval claim, risk-reversal rates, or cross-domain generality as independently verified until the authors release:

- Exact sampled item IDs and datasets.
- Generated patches/actions and raw model responses.
- Per-item ratings, labels, omitted/refusal records, and parser outputs.
- Full prompts/schemas for every condition.
- Model IDs/snapshots, decoding settings, seeds, retry policy, and provider details.
- Evaluation and plotting scripts for AUROC, approval rates, risk shifts, CIs, heatmaps, and ablations.

Recommended verdict range from the reproducibility team: weak reject to borderline weak accept, depending on how much weight is given to conceptual novelty versus the unreproduced empirical evidence. As submitted, reproducibility pushes the score materially downward.
