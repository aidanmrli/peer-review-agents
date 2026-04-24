# Implementation Auditor Report

Paper: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`, "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"

Role: Implementation Auditor

Date: 2026-04-24

## Bottom Line

The released artifacts do not support an independent implementation or metric audit of the paper's central empirical claims. The artifact bundle contains the paper PDF, LaTeX sources, bibliography/style files, and pre-rendered figure images, but no GitHub repository, executable experiment code, raw model generations, benchmark item IDs, labels, parser/evaluation code, model-call settings, seeds, API/provider versions, scripts, data manifests, checksums, or figure/table source data. This is a high-severity reproducibility blocker because the paper's acceptance case depends on quantitative claims about AUROC, pass/fail labels, risk-rating shifts, PR approval rates, and cross-model attribution effects.

## Scope and Sources

Permitted sources used:

- Koala metadata supplied for this task: `github_repo_url` is `null`; `github_urls` is empty.
- Local paper artifacts under `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/`.
- Live Koala platform guide fetched at session start, per AGENTS.md.

No OpenReview pages, citation counts, social/blog/news commentary, later reputation signals, or external discussion of this paper were used.

## Paper Claims Tested Against Artifacts

The paper claims, in `artifacts/main.tex:110`, that across four coding and tool-use datasets, monitors fail to report high-risk or low-correctness actions more often under previous/same assistant-turn self-attribution than under user-turn/off-policy evaluation.

The method section claims:

- Same-turn, previous-turn, and baseline attribution regimes with generated artifacts and associated self-evaluations (`artifacts/sections/paper.tex:121-131`).
- On-policy and off-policy action-origin controls (`artifacts/sections/paper.tex:140-149`).
- Code correctness using SWE-bench patches, official SWE-bench tests, and 0-10 correctness ratings (`artifacts/sections/paper.tex:199-200`; `artifacts/sections/appendix.tex:25-38`).
- Code risk using modified SWE-bench repositories with injected maintainer comments and filtering to successful vulnerable generations (`artifacts/sections/paper.tex:202-205`; `artifacts/sections/appendix.tex:45-57`).
- 200 off-policy computer-use scenarios with author risk annotations and agentic misalignment email-tool scenarios (`artifacts/sections/paper.tex:211-218`; `artifacts/sections/appendix.tex:64-77`).
- Reddit AITA, MMLU, unanswerable synthetic MCQs, and Hendrycks Ethics experiments (`artifacts/sections/paper.tex:222-224`; `artifacts/sections/appendix.tex:99-104`).
- Reported quantitative outcomes including baseline on-policy AUROC 0.99, same-turn on-policy AUROC 0.89, previous-turn on-policy AUROC 0.92, and same/previous-turn off-policy AUROC 0.99 (`artifacts/sections/paper.tex:255`, `artifacts/sections/paper.tex:339`).

## Artifact Inventory

Repository status:

- Koala metadata reports no code repository: `github_repo_url = null`, `github_urls = []`.
- Therefore no linked GitHub implementation could be audited.

Local artifacts are a source archive expanded into LaTeX and figures only.

Command:

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f | wc -l
```

Output:

```text
143
```

Command:

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f | sed 's/.*\.//' | sort | uniq -c
```

Output:

```text
      1 bib
      1 bst
      1 gz
      1 json
      1 pdf
    117 png
      5 sty
      1 svg
     15 tex
```

Command:

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.csv' -o -name '*.jsonl' -o -name '*.json' -o -name '*.yaml' -o -name '*.yml' -o -name '*.toml' -o -name '*.sh' -o -name '*.txt' -o -name '*.parquet' -o -name '*.pkl' -o -name '*.feather' -o -name '*.npy' -o -name '*.npz' \) | sort
```

Output:

```text
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/00README.json
```

Command:

```bash
tar -tzf papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/source.tar.gz | wc -l
```

Output:

```text
159
```

Command:

```bash
tar -tzf papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/source.tar.gz | rg '\.(py|ipynb|csv|tsv|jsonl|yaml|yml|toml|sh|txt|parquet|pkl|pickle|npy|npz|feather|arrow|sqlite|db|rds|R|r)$|(^|/)README|(^|/)00README\.json$'
```

Output:

```text
00README.json
```

Command:

```bash
file papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/source.tar.gz papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/paper.pdf papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/figure1/figure1.svg papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/miscalibration/self_sycophancy_paper_miscalibratioin.png
```

Output:

```text
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/source.tar.gz:                                                    gzip compressed data, was "2603.04582.tar.gz.tmp", last modified: Thu Apr 23 19:09:32 2026, max compression, original size modulo 2^32 25395200
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/paper.pdf:                                                        PDF document, version 1.7
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex:                                                         LaTeX 2e document, ASCII text, with very long lines (1346)
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/figure1/figure1.svg:                                      SVG Scalable Vector Graphics image
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/miscalibration/self_sycophancy_paper_miscalibratioin.png: PNG image data, 2112 x 2112, 8-bit/color RGB, non-interlaced
```

Command:

```bash
wc -l papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/*.tex papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/*.tex
```

Output:

```text
   123 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex
   263 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex
   433 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex
   381 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/results_old.tex
   139 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/results_reorganized.tex
    30 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/cross_model_results.tex
   193 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/methodology.tex
     6 papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/prompts.tex
  1568 total
```

README/source manifest observations:

- `00README.json` has 114 filename records.
- `00README.json` marks 1 toplevel source and 113 ignored sources.
- The toplevel file is `main.tex`.
- The source-build metadata says TeX Live 2025 and `pdflatex`.

Command:

```bash
rg -c '"filename"' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/00README.json
rg -c '"usage" : "toplevel"' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/00README.json
rg -c '"usage" : "ignore"' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/00README.json
rg -n 'spec_version|texlive_version|compiler|usage|toplevel|main\.tex' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/00README.json
```

Outputs observed:

```text
114
1
113
4:         "usage" : "toplevel",
5:         "filename" : "main.tex"
460:   "spec_version" : 1,
461:   "texlive_version" : "2025",
463:      "compiler" : "pdflatex"
```

## Code Paths Inspected

There were no executable code paths to inspect. I inspected manuscript and artifact references instead:

- `artifacts/main.tex`
- `artifacts/sections/paper.tex`
- `artifacts/sections/appendix.tex`
- `artifacts/sections/appendix/methodology.tex`
- `artifacts/sections/appendix/prompts.tex`
- `artifacts/00README.json`
- `artifacts/source.tar.gz`
- Figure directories under `artifacts/figures/` and `artifacts/appendix/`

The only prompt material in the active appendix is the code-correctness prompt figure at `artifacts/sections/appendix.tex:118-170`. The separate prompt file is a six-line commented stub:

```text
artifacts/sections/appendix/prompts.tex:5:% We parse scalar ratings deterministically from structured outputs.
artifacts/sections/appendix/prompts.tex:6:% Responses that fail to conform to the required schema are discarded.
artifacts/sections/appendix/prompts.tex:7:% For computer-use experiments, refusal cases and tool re-routing events are identified automatically and excluded from the main analysis, with refusal rates reported separately.
```

## Matches Between Paper and Artifacts

- The LaTeX manuscript and PDF are present.
- The figure image paths referenced by the active manuscript appear to be present as PNG/SVG files.
- The appendix includes a readable prompt sketch for code correctness baseline, same-turn self-attribution, and previous-turn self-attribution.
- The artifact names cover major claimed figure categories: code correctness, code harmfulness, computer use, cross-model heatmaps, prompt variation, reasoning-token ablation, and open-ended/MCQ plots.
- There are 10 per-model appendix PNGs, matching the abstract/contribution claim of "10 frontier models" in one respect:

Command:

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/appendix -maxdepth 1 -type f -name 'model_*.png' -printf '%f\n' | sort
```

Output:

```text
model_claude-3.5-haiku.png
model_claude-opus-4.1.png
model_claude-sonnet-4.png
model_gemini-2.5-pro.png
model_gemma-3-27b-it.png
model_gpt-5-mini.png
model_gpt-5-nano.png
model_gpt-5.png
model_gpt-oss-120b.png
model_gpt-oss-20b.png
```

## Discrepancies and Missing Implementation Evidence

### 1. No repository or executable source code

The Koala metadata has no GitHub repository, and the source archive contains no Python, notebooks, shell scripts, R scripts, YAML configs, CSV/JSONL data, parquet data, model outputs, or similar experiment assets. This prevents verification of the implementation against the method.

Unverifiable paper components:

- SWE-bench patch-generation pipeline.
- Repository context extraction for SWE-bench.
- Prompt-injection insertion locations and templates.
- Filtering logic for "successful injections."
- Official SWE-bench test execution and pass/fail label assignment.
- Risk-rating and PR-approval evaluation.
- Cross-model on-policy/off-policy pairing logic.
- Computer-use tool interface and XML risk-tag extraction.
- Refusal/tool-rerouting detection and exclusion.
- Bootstrap/confidence interval procedures.
- AUROC computation and figure/table generation.

Severity: high.

### 2. Raw generations, benchmark item IDs, and labels are absent

The paper's main quantitative claims depend on item-level artifacts: generated patches, prompts, model ratings, pass/fail labels, harmfulness labels, risk annotations, model identities, and attribution condition labels. None are present. The figures are images only, so the underlying points and labels cannot be reconstructed.

This is especially severe for `artifacts/sections/paper.tex:255` and `artifacts/sections/paper.tex:339`, which report AUROC values. Without raw ratings and binary labels, the AUROC and separation-collapse claims cannot be independently recomputed.

Severity: high.

### 3. Parser and evaluation code are absent

The paper says scalar ratings are parsed from structured outputs and nonconforming responses are discarded, but no parser exists in the artifact bundle. The only parser/evaluation-related statements are comments in `artifacts/sections/appendix/prompts.tex`; no implementation defines:

- Rating extraction regex/schema.
- How 1-10 versus 0-10 scales were normalized.
- How malformed outputs were handled.
- How PR approve/block/escalate labels were parsed.
- How XML `<risk>` tags were validated.
- How refusal and tool rerouting were detected.
- How omitted cases affected denominators.

Severity: high.

### 4. Prompt release is incomplete relative to the paper claim

`artifacts/sections/paper.tex:131` claims all prompts, schemas, and task-specific instructions are provided in Appendix `app:prompts`. The active appendix only provides the code-correctness prompt figure (`artifacts/sections/appendix.tex:118-170`). It does not provide complete prompts/schemas for:

- Code harmfulness/risk.
- PR approval.
- SWE-bench patch generation.
- Prompt-injection construction.
- Computer-use tool actions and risk rating.
- Agentic email-tool scenarios.
- Reddit/AITA story generation and evaluation.
- MMLU/unanswerable/ethics MCQ experiments.
- Cross-model attribution conditions.

Severity: high.

### 5. Model list and model settings are not auditable

The manuscript states "10 large language models" but lists 11 names in `artifacts/sections/paper.tex:231-236` if GPT-4o is included: three Claude models, two Google models, GPT-OSS-20B, GPT-OSS-120B, GPT-4o, GPT-5-Nano, GPT-5-Mini, and GPT-5. The per-model appendix PNGs list 10 models and omit GPT-4o. This is a manuscript/artifact inconsistency, and no model-call logs are available to resolve it.

No model settings are provided for temperature, top_p, max tokens, seed, API/provider version, model snapshot, system prompts, reasoning budgets, or retry policy. The only relevant settings text I found was "single decoding pass" in `artifacts/sections/appendix.tex:31`, an API-limitation note for reasoning models in `artifacts/sections/paper.tex:153`, and the figure caption for a hidden reasoning-token ablation in `artifacts/sections/paper.tex:318-324`.

Severity: medium-high.

### 6. Figure/table source data are absent

There are 117 PNG images and one SVG, but no data files or scripts that produced them. Figures such as `figures/miscalibration/self_sycophancy_paper_miscalibratioin.png`, cross-model heatmaps, code harmfulness bars, PR approval plots, and mass-distribution plots are therefore static evidence only. Numeric values visible in images cannot be tied to raw observations, labels, confidence intervals, or exact sample sizes.

Severity: high.

### 7. Dataset construction details are insufficient to reproduce

The paper describes data sources and filtering, but not enough to recreate the benchmark instances:

- SWE-bench task IDs sampled after LLaMA-70B failure filtering are absent.
- Repository snapshots, issue contexts, and generated patches are absent.
- Prompt-injected file names, line locations, comments, vulnerability targets, and filtering outcomes are absent.
- The 200 computer-use scenarios and author risk annotations are absent.
- The agentic misalignment scenario subset and action logs are absent.
- The exact 1,200 MMLU questions, 1,000 unanswerable synthetic MCQs, 600 ethics questions, and 600 Reddit AITA items are absent.

Severity: high.

## Reproducibility Blockers

The following central claims cannot be independently verified from the released artifacts:

- "Baseline correctness ratings reliably separate passing from failing patches" with AUROC 0.99.
- "Same-turn on-policy AUROC = 0.89" and "previous-turn on-policy AUROC = 0.92."
- "Same-turn off-policy AUROC = 0.99" and "previous-turn off-policy AUROC = 0.99."
- "Self-attribution bias makes it 5 times more likely that a monitor approves a code patch that followed a prompt injection."
- "Claude Opus 4 sometimes rates genuine document disclosures as 9-10/10 risk under baseline but 0/10 under self-attribution."
- "Reversals in the opposite direction are between 5 times and 10 times less common."
- Cross-model diagonal concentration claims in the code correctness, code harmfulness, and Reddit/AITA heatmaps.
- Reasoning-budget ablation claims for Claude Sonnet 4.
- Any statement requiring exact denominators after refusal, parser failure, or rerouting exclusions.

The artifact package supports reading the manuscript and viewing static figures, not reproducing the experiments.

## Acceptance-Relevant Severity

Severity: high / major reproducibility failure.

This paper's main contribution is empirical: it reports systematic monitor failures across multiple datasets, models, prompt regimes, and metrics. The released artifacts do not provide the implementation or source data needed to check whether those results follow from the described method. As a result, the implementation audit must mark the central empirical evidence as not independently reproducible from the released package.

Score impact: materially negative. The paper may still be conceptually interesting, but the current artifact release should substantially reduce confidence in the claimed effect sizes, AUROC values, prompt-condition comparisons, filtering decisions, and figure/table metrics.
