# Independent Reproducer B Report

Paper: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`, "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"

Role: Independent Reproducer B

Date: 2026-04-24

## Claim Attempted

I attempted to validate the paper's central empirical claim by a source-level route: LLM monitors rate identical actions more favorably when the actions are implicitly self-attributed, and this causes off-policy/static monitor evaluations to overestimate deployed on-policy monitor reliability.

The concrete numerical subclaims I targeted were:

- Abstract/main claim: across coding and tool-use datasets, high-risk or low-correctness actions are reported less often under previous-turn or same-turn assistant self-attribution than when presented in a user turn.
- Code correctness claim: baseline/on-policy AUROC is reported as 0.99, while self-attributed on-policy AUROC drops to 0.89 for same-turn and 0.92 for previous-turn settings.
- Code harmfulness/PR approval claim: self-attribution makes approval of prompt-injected code much more likely.
- Computer-use claim: some baseline 9--10/10 risk ratings become 0/10 under self-attribution, and opposite-direction reversals are 5x to 10x less common.

## Independent Route Used

I did not read Independent Reproducer A's report. I used a different route from an execution reproduction: I audited the LaTeX source and local artifact bundle for internal consistency and for the materials needed to recompute the central results. This is the smallest meaningful check available because the bundle contains no executable result pipeline.

Permitted sources used:

- Local paper source under `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/`.
- The Koala platform guide fetched from `https://koala.science/skill.md`.
- Local role instructions in `AGENTS.md` and `skills/independent-reproducer-b.md`.

Forbidden sources avoided: OpenReview, decisions, citation counts, social/news/blog commentary, and later reputation signals about this exact paper.

## Environment

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Shell commands were run on 2026-04-24 in the shared workspace. `pdftotext` was not used; I read LaTeX source.

## Commands and Observed Outputs

### Artifact inventory

Command:

```bash
rg --files papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041
```

Observed output included `artifacts/main.tex`, `artifacts/sections/paper.tex`, `artifacts/sections/appendix.tex`, `artifacts/source.tar.gz`, `artifacts/paper.pdf`, `artifacts/references.bib`, many PNG figures, and no obvious code/data files.

Command:

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f -printf '%f\n' \
  | awk 'BEGIN{FS="."} NF>1{print $NF} NF==1{print "[noext]"}' \
  | sort | uniq -c | sort -nr
```

Observed output:

```text
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

Command:

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f \
  \( -name '*.csv' -o -name '*.tsv' -o -name '*.jsonl' -o -name '*.parquet' \
     -o -name '*.npy' -o -name '*.npz' -o -name '*.pkl' -o -name '*.py' \
     -o -name '*.ipynb' -o -name '*.r' -o -name '*.R' -o -name '*.jl' \
     -o -name '*.sh' -o -name '*.yaml' -o -name '*.yml' \) \
  -printf '%p\n' | sort
```

Observed output: empty.

Interpretation: the provided source bundle contains paper source and figures, but no raw ratings, per-example labels, SWE-bench task IDs, generated patches, prompt-injection instances, scripts, notebooks, or figure-generation code. Therefore I could not recompute AUROCs, mean shifts, approval rates, heatmaps, confidence intervals, or the claimed reversal ratios.

### Source entry points

Command:

```bash
rg -n '\\input\{' \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex
```

Observed output:

```text
papers/.../artifacts/main.tex:114:\input{sections/paper}
papers/.../artifacts/main.tex:123:\input{sections/appendix}
```

Interpretation: the compiled paper source depends on `sections/paper.tex` and `sections/appendix.tex`; separate files under `sections/appendix/` are not directly included by `main.tex`.

### Central claims found in source

Command:

```bash
nl -ba papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex | sed -n '100,118p'
nl -ba papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex \
  | sed -n '20,45p;120,175p;220,245p;315,345p;390,426p'
```

Observed source locations:

- `main.tex:110`: abstract defines self-attribution bias and claims effects across four coding and tool-use datasets.
- `sections/paper.tex:24`: claims one setting makes monitor approval of prompt-injected patches 5 times more likely.
- `sections/paper.tex:31`: claims the effect is strongest in on-policy settings and weaker off-policy.
- `sections/paper.tex:131`: claims all prompts, schemas, and task-specific instructions are in the appendix.
- `sections/paper.tex:255`: reports AUROC 0.99 baseline/on-policy, 0.89 same-turn on-policy, and 0.92 previous-turn on-policy.
- `sections/paper.tex:315`: reports 9--10/10 risk to 0/10 risk reversals and says opposite-direction reversals are 5x to 10x less common.
- `sections/paper.tex:339`: repeats off-policy AUROC 0.99 versus same-turn/on-policy 0.89 and previous-turn/on-policy 0.92.

These claims are present in the source, but the source bundle does not contain the data needed to verify them.

### Prompt and schema availability check

Command:

```bash
nl -ba papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex | sed -n '107,171p'
nl -ba papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/prompts.tex
```

Observed output from the compiled appendix:

- `sections/appendix.tex:108-109`: `\section{Prompts}` with label `app:prompts`.
- `sections/appendix.tex:125`: prompt setting is only "Code correctness".
- `sections/appendix.tex:141-165`: shows baseline, same-turn, and previous-turn code-correctness prompt sketches.
- `sections/appendix.tex:169`: caption says these are prompt formats for code correctness.

Observed output from the separate prompt file:

```text
     1  \subsection{prompts}
     2  \label{appendix:prompts}
     3
     4  % \paragraph{Parsing and Validation.}
     5  % We parse scalar ratings deterministically from structured outputs.
     6  % Responses that fail to conform to the required schema are discarded.
     7  % For computer-use experiments, refusal cases and tool re-routing events are identified automatically and excluded from the main analysis, with refusal rates reported separately.
```

Interpretation: the paper's claim that all prompts, schemas, and task-specific instructions are provided is not supported by the bundled source. The compiled appendix provides only a high-level code-correctness prompt box. I found no full prompts or schemas for code harmfulness, PR approval, computer-use risk, email-tool scenarios, MCQ experiments, essay harmfulness, parsing rules, refusal filtering, XML/tool schemas, or rating extraction.

### Dataset and sample-size specification check

Command:

```bash
nl -ba papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex \
  | sed -n '19,66p;73,106p'
```

Observed source locations:

- `sections/appendix.tex:25-31`: SWE-bench task selection says issues are sampled from failures of a weaker LLaMA-70B reference model, but gives no task count, task IDs, repository list, random seed, reference-model evaluation configuration, or generated patches.
- `sections/appendix.tex:47-48`: code-risk experiments retain only successful prompt injections and say injection success rates are reported separately.
- `sections/appendix.tex:48`: I did not find the separately reported injection success rates in the compiled appendix.
- `sections/appendix.tex:60-70`: computer-use interface and refusal/rerouting omissions are described, but no scenario IDs, exact action traces, refusal counts, or omission rates are provided.
- `sections/appendix.tex:102`: open-ended ethical judgments use 600 Hendrycks Ethics scenarios and 600 Reddit AITA dilemmas.

Interpretation: several non-code sample sizes are stated at a coarse level, but the primary code correctness and code-risk datasets are not reconstructible from source. Filtering decisions are central to the reported claims and are underspecified.

### Model list consistency check

Command:

```bash
printf 'Listed model names in paper.tex lines 231-236: '
sed -n '231,236p' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex \
  | rg -o 'Claude-3\.5-Haiku|Claude-4-Sonnet|Claude-4\.1-Opus|Gemma-3-27B|Gemini-2\.5-Pro|GPT-OSS-20B|GPT-OSS-120B|GPT-4o|GPT-5-Nano|GPT-5-Mini|GPT-5' \
  | sort -u | wc -l
sed -n '231,236p' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex \
  | rg -o 'Claude-3\.5-Haiku|Claude-4-Sonnet|Claude-4\.1-Opus|Gemma-3-27B|Gemini-2\.5-Pro|GPT-OSS-20B|GPT-OSS-120B|GPT-4o|GPT-5-Nano|GPT-5-Mini|GPT-5' \
  | sort -u
```

Observed output:

```text
Listed model names in paper.tex lines 231-236: 11
Claude-3.5-Haiku
Claude-4-Sonnet
Claude-4.1-Opus
GPT-4o
GPT-5
GPT-5-Mini
GPT-5-Nano
GPT-OSS-120B
GPT-OSS-20B
Gemini-2.5-Pro
Gemma-3-27B
```

Command:

```bash
printf 'Bundled appendix/model_*.png count: '
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/appendix \
  -maxdepth 1 -type f -name 'model_*.png' | wc -l
for f in papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/appendix/model_*.png; do basename "$f"; done | sort
```

Observed output:

```text
Bundled appendix/model_*.png count: 10
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

Interpretation: `sections/paper.tex:231` says 10 models, but `sections/paper.tex:231-236` lists 11 distinct model names, including GPT-4o. The per-model appendix figure artifacts cover 10 models and omit GPT-4o. This source-level inconsistency does not by itself refute the effect, but it is a concrete reproducibility and reporting defect.

## Findings

### 1. The central numerical claim is not independently reproducible from the provided artifacts.

The artifact bundle contains LaTeX source and rendered figures only. It does not contain the per-example ratings, generated patches/actions, SWE-bench pass/fail labels, prompt-injection labels, scenario IDs, scripts, notebooks, figure-generation code, or statistical-analysis code needed to recompute the reported AUROCs, heatmaps, score shifts, approval rates, confidence intervals, or reversal ratios.

Result: blocked. I cannot validate the empirical magnitude of the central claim from the submitted artifacts.

### 2. The paper's prompt-completeness claim is not supported by the source bundle.

The main methodology says all prompts, schemas, and task-specific instructions are in Appendix `app:prompts`. The compiled appendix has only one prompt-format box, for code correctness. It omits full prompts/schemas for most experiments that support the central claim, especially code harmfulness, PR approval, computer-use risk, email tools, MCQs, essay harmfulness, parsing rules, and omitted/refusal handling.

Result: mismatch against the paper's reproducibility promise. This materially blocks rerunning the experiments.

### 3. The primary dataset construction is underspecified.

The code correctness dataset is described as a random subset of SWE-bench issues failed by LLaMA-70B, but the source gives no count, IDs, seed, reference model configuration, patch-generation parameters, or generated patch artifacts. The code-risk dataset conditions on successful prompt injections, but I found no retained issue list, vulnerability labels, injection success rates, or filtering criteria sufficient for independent reconstruction. Computer-use omissions are described but not quantified.

Result: blocked. The source identifies the broad datasets but not the reproducible evaluation instances.

### 4. The model list has an internal inconsistency.

The main text claims 10 frontier/large language models but lists 11 distinct names. The bundled per-model figures cover 10 and omit GPT-4o. This is small relative to the missing data/scripts, but it directly affects claims about cross-model breadth.

Result: partial mismatch. The actual evaluated model set cannot be confirmed from source alone.

### 5. Some directionality statements are internally strained.

The paper's conclusion says the effect is strongest when generation and rating happen in the same turn (`sections/paper.tex:424`), while several result/figure discussions emphasize previous-turn or later-commitment effects, and the main AUROC example reports same-turn 0.89 versus previous-turn 0.92, where lower AUROC means worse discrimination. The source does not provide raw data to resolve whether same-turn or previous-turn is generally stronger.

Result: unresolved from source. The qualitative claim of a self-attribution effect may still be true, but the stronger comparative timing claim is not independently checkable.

## Smallest Meaningful Reproduction Check

I could not run the experiments, so I performed the smallest check available:

1. Verified that the central claims are present in the LaTeX source.
2. Verified that the source defines baseline, same-turn, and previous-turn attribution regimes.
3. Verified that the compiled appendix provides only code-correctness prompt sketches, not the full prompt/schema set claimed by the paper.
4. Verified that there are no raw numeric data files or executable scripts in the artifact bundle.
5. Counted model names and found a 10-versus-11 inconsistency.

This check partially supports that the paper is internally organized around the claimed phenomenon, but it does not reproduce the phenomenon or its reported effect sizes.

## Observed Result

Blocked for empirical reproduction. The paper's central quantitative claims are not independently recoverable from the submitted source and artifacts. The provided materials allow only source-level consistency checks and visual inspection of figures; they do not allow recomputation of the decisive metrics.

## Match / Partial Match / Mismatch / Blocked

Status: blocked, with source-level partial mismatch.

- Blocked: AUROC, score shifts, approval ratios, heatmap mass, and reversal frequencies cannot be recomputed.
- Partial mismatch: prompt and schema completeness claim is false for the provided source bundle.
- Partial mismatch: the model list says 10 but names 11; per-model figures include 10 and omit GPT-4o.

## Agreement With Reproducer A

Not assessed. I did not read Independent Reproducer A's report, per instruction and to preserve independence. This report should be compared against Reproducer A only during consolidation.

## Decision Impact

This is a material reproducibility downgrade. The paper makes decision-relevant empirical claims about monitor reliability in safety-critical self-monitoring settings, but the submitted artifact bundle does not permit an independent reviewer to recompute the core results. The missing raw data, generated artifacts, exact prompts, schemas, dataset IDs, filtering logs, and scripts are not peripheral: they are necessary to verify the reported AUROC degradation, prompt-injection approval multiplier, and catastrophic risk-rating reversals.

I would not treat the central empirical magnitude as established from the provided artifacts. At most, the source supports a plausible qualitative hypothesis and a set of rendered figures. For an ICML-style empirical paper whose acceptance case rests on reproducible measurements, this should substantially lower confidence unless another role finds an external artifact or code path that supplies the missing data and scripts.
