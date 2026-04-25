# Independent Reproducer A Report

## Paper

- Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`
- Title: `Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes`
- Assigned role: Independent Reproducer A
- Date: 2026-04-24

## Task Scope

I independently attempted to verify the central empirical and computational claims from the paper text and official Koala artifacts only. I did not read Independent Reproducer B's report. The main question tested was whether Delta-Crosscoder can be independently reproduced from the supplied paper package well enough to validate its core claims:

1. Delta-Crosscoder reliably identifies causal fine-tuning-induced latents across 10 model organisms.
2. Steering or ablating those latents induces, suppresses, or mitigates the target behaviors.
3. Delta-Crosscoder outperforms SAE-based crosscoder baselines and approximately matches the non-SAE ADL baseline.
4. The method can be implemented from the paper and artifact details.

## Evidence Examined

- Koala paper metadata via `get_paper`:
  - `status`: `in_review`
  - `arxiv_id`: `2603.04426`
  - `github_urls`: `[]`
  - `github_repo_url`: `null`
  - `pdf_url`: `/storage/pdfs/4ce90b72-2181-4118-aa61-b80b9acbbcce.pdf`
  - `tarball_url`: `/storage/tarballs/4ce90b72-2181-4118-aa61-b80b9acbbcce.tar.gz`
- Local artifact directory:
  - `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex`
  - `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.bib`
  - `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/paper.pdf`
  - `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/source.tar.gz`
  - Five figure PDFs under `artifacts/figures/`
  - ICML style files and `00README.json`
- No external exact-paper sources, OpenReview material, citation counts, social media, later outcome signals, or future information were used.

## Setup Used

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Tool/environment details observed:

```text
python 3.12.12
platform Linux-5.15.0-173-generic-x86_64-with-glibc2.35
rg ripgrep 15.1.0
tar GNU tar 1.34
pdftotext unavailable
pdfinfo unavailable
```

The absence of `pdftotext` and `pdfinfo` did not block the review because the LaTeX source was available and was the more precise source for formulas, tables, and artifact references.

## Commands and Checks

### Artifact inventory

```bash
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -maxdepth 4 -type f | sort
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -maxdepth 4 -type f | wc -l
du -ah papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts | sort -h | tail -30
tar -tzf papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/source.tar.gz | sort | sed -n '1,240p'
```

Observed result:

- The artifact directory contains 16 files, about 14 MB total.
- The tarball contains only TeX source, bibliography, style files, and five PDF figures.
- `00README.json` identifies `example_paper.tex` as the top-level source and `pdflatex` with TeXLive 2025 as the build process.

### Code/config/data/checkpoint search

```bash
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -type f \
  \( -name '*.py' -o -name '*.ipynb' -o -name '*.json' -o -name '*.yaml' \
     -o -name '*.yml' -o -name '*.toml' -o -name '*.csv' -o -name '*.tsv' \
     -o -name '*.npz' -o -name '*.pt' -o -name '*.safetensors' \
     -o -name '*.ckpt' -o -name '*.log' -o -name '*.txt' \) -print | sort
```

Observed result:

```text
papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/00README.json
```

Interpretation: apart from the TeX build manifest, there is no executable code, notebook, machine-readable config, raw table data, generated sample batch, model checkpoint, crosscoder checkpoint, activation cache, training log, evaluation log, or metric output file in the official artifacts.

### Paper-source search for central claims and reproducibility details

```bash
rg -n "Delta|Crosscoder|crosscoder|narrow|result|Table|Figure|dataset|Qwen|organism|refusal|steering|agent|GitHub|code|repository|artifact|seed|checkpoint|hyperparameter|baseline" \
  papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex
```

Key locations:

- Abstract and central claim: `example_paper.tex:191`
- Contributions: `example_paper.tex:232-240`
- Crosscoder preliminaries: `example_paper.tex:273-289`
- Relative decoder norm: `example_paper.tex:291-301`
- Delta loss: `example_paper.tex:310-333`
- Contrastive text pairs: `example_paper.tex:348-357`
- Dual-K masking and objective: `example_paper.tex:359-390`
- Model organisms: `example_paper.tex:397-415`
- Training data: `example_paper.tex:505-519`
- Evaluation methodology: `example_paper.tex:536-549`
- Main organism results: `example_paper.tex:596-741`
- Baseline comparisons: `example_paper.tex:745-801`
- False-positive/null-test claims: `example_paper.tex:811-831`
- Hyperparameter table: `example_paper.tex:884-923`
- Reconstruction/dead-feature table: `example_paper.tex:927-1013`
- Steering procedure: `example_paper.tex:1015-1056`
- SDF setup: `example_paper.tex:1059-1069`
- Ablations: `example_paper.tex:1100-1138`
- Persona-vector comparison: `example_paper.tex:1140-1157`

### Clean-room table arithmetic check

I parsed the appendix metric table and recomputed `dead / dictionary_size * 100`, rounded to one decimal:

```bash
python3 - <<'PY'
import re, pathlib
p=pathlib.Path('papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex')
tex=p.read_text()
rows=[]
for line in tex.splitlines():
    if '&' in line and '\\\\' in line and not any(x in line for x in ['textbf','multicolumn']):
        clean=re.sub(r'\\\\.*','',line).strip()
        parts=[re.sub(r'\\\\[^{}]+\\{([^}]*)\\}', r'\\1', x).strip() for x in clean.split('&')]
        if len(parts)==6:
            try:
                method=parts[1]
                dict_size=int(parts[2])
                dead=int(parts[4])
                dead_pct=float(parts[5])
                calc=round(dead/dict_size*100,1)
                rows.append((method,dict_size,dead,dead_pct,calc,dead_pct-calc))
            except Exception:
                pass
print(f'parsed_rows={len(rows)}')
for r in rows:
    method,dict_size,dead,reported,calc,diff=r
    flag='OK' if abs(diff) < 0.06 else 'MISMATCH'
    print(f'{method:8s} dict={dict_size:5d} dead={dead:5d} reported={reported:4.1f} calc={calc:4.1f} {flag}')
PY
```

Observed result:

- Parsed rows: 32.
- 29 of 32 dead-rate percentages match the reported one-decimal values.
- Three one-decimal discrepancies:
  - Qwen EM (Extreme Sports), TopK-200: `2319 / 17920 * 100 = 12.9`, reported `13.0`.
  - Qwen EM (Extreme Sports), TopK-400: `11534 / 17920 * 100 = 64.4`, reported `64.3`.
  - Qwen Subliminal, TopK-400: `10367 / 17920 * 100 = 57.9`, reported `57.8`.

These discrepancies are numerically small and not decision-critical. The table arithmetic mostly matches, but this check only validates simple reported percentages, not the underlying training runs.

## Findings

### 1. Full empirical reproduction is blocked

The central empirical claim cannot be independently rerun from the supplied artifacts. The paper claims broad causal recovery across 10 model organisms and multiple model families (`example_paper.tex:191`, `234`, `397-415`, `596-741`), but the artifact package provides no runnable implementation or experimental records.

Concretely missing:

- Delta-Crosscoder training code.
- Baseline training code for DSF, BatchTopK-200, BatchTopK-400, and ADL-comparison preparation.
- Exact model checkpoint identifiers for every base and finetuned organism.
- Fine-tuned organism checkpoints or adapters.
- Crosscoder checkpoints or learned latent dictionaries.
- Activation extraction code and cached activations.
- Exact layer choices per model, beyond the generic statement "middle layer" (`example_paper.tex:418-419`).
- Exact sampling manifests for FineWeb, LMSYS, finetuning data, and contrastive prompts (`example_paper.tex:505-519`).
- Generation parameters for constructing the 200,000 contrastive prompt-response pairs.
- Steering implementation code and the model-specific normalization factors mentioned in `example_paper.tex:1042-1047`.
- Max-activation examples as machine-readable data, beyond selected figure text.
- Judge prompts, grader outputs, and rubric implementation for the GPT-5.2 comparison to ADL (`example_paper.tex:779-801`).
- Seeds, hardware details, optimizer betas, scheduler details beyond warmup count, logging, and failure handling.

The Koala metadata also reports no linked GitHub repository (`github_urls: []`, `github_repo_url: null`). This is a decisive reproduction blocker for a computational interpretability paper whose claims depend on training large dictionaries and evaluating causal steering.

### 2. The method is only partially implementable from the paper

The high-level math is mostly reconstructible:

- A standard crosscoder encodes base and fine-tuned activations and reconstructs both models (`example_paper.tex:277-289`).
- Relative decoder norm is defined as a post-hoc specificity statistic (`example_paper.tex:291-301`).
- Delta-Crosscoder adds an auxiliary difference-reconstruction loss (`example_paper.tex:310-333`).
- Shared latents are masked for the delta loss (`example_paper.tex:359-380`).
- The full loss is written as reconstruction plus sparsity plus weighted delta loss (`example_paper.tex:382-390`).

However, the paper-level specification is not sufficient for faithful implementation. The equations omit or leave ambiguous several implementation choices that directly affect the result:

- The exact BatchTopK implementation and Dual-K allocation are underspecified. The method says `K_delta = alpha * K_shared` with `alpha < 1` (`example_paper.tex:364-365`), but the hyperparameter table lists base sparsity, shared-k multiplier, shared feature fraction, and `AuxK Coefficient`; it does not clearly state the actual `alpha` used for `K_delta` (`example_paper.tex:892-923`).
- The loss includes a "sparsity regularizer implemented" phrase (`example_paper.tex:382`), but BatchTopK sparsity normally replaces an explicit sparsity penalty; the exact loss term used in training is not recoverable.
- The method says the activation difference "does not require" matched inputs (`example_paper.tex:316`), while the contrastive construction later uses paired base/finetuned activations from shared generated text (`example_paper.tex:348-357`). The exact rule for when delta loss is applied to paired versus unpaired activations is not specified.
- The training mixture among pretraining-style text, instruction data, finetuning data, and contrastive data is described qualitatively but not as an exact sampling schedule (`example_paper.tex:505-519`).
- The steering formula is given, but not enough information is supplied to reproduce model interventions because the residual-stream hook location, decoder-vector orientation, and normalization factors are not released (`example_paper.tex:1017-1056`).

Result: a reader could implement a plausible Delta-Crosscoder variant, but not the exact submitted experiment.

### 3. Only a small arithmetic check was reproduced

The appendix dead-feature percentages in Table `tab:metrics` are mostly internally consistent with the dead counts and dictionary sizes. This is a successful reproduction of a minor reporting calculation only. It does not validate the central model-diffing, steering, baseline, or robustness claims.

Additional table observations:

- The metric table contains 8 organism rows, while the paper repeatedly claims evaluation over 10 model organisms (`example_paper.tex:397`, `596`, `753`, `819`). The remaining organisms may be represented only in figures or prose, but raw per-organism tables or logs are not supplied.
- Several rows repeat identical explained-variance values across different organisms and methods, for example `80.46`, `80.07`, `79.29`, and `81.64` recur across LLaMA EM settings (`example_paper.tex:965-980`). This is not an arithmetic contradiction, but without logs it cannot be distinguished from rounded reuse, shared training runs, or copied summary values.

### 4. Evaluation claims are not recoverable from figures alone

The figures are static PDFs:

- `figures/Organism_Coverage_Comparison.pdf`
- `figures/agents_comp.pdf`
- `figures/qwen_com.pdf`
- `figures/refusal_latent.pdf`
- `figures/steering_results.pdf`

No raw figure data or plotting scripts are included. I could not recover underlying counts, prompt-level responses, judge scores, or confidence intervals from the artifacts. The steering-response figure gives qualitative examples, but it does not substitute for reproducible response generation or scoring.

### 5. The empirical protocol needs substantial hidden state

The paper's core evidence depends on expensive and stateful computations:

- Running base and finetuned LLMs from Gemma, LLaMA, and Qwen families.
- Generating 200,000 contrastive prompt-response pairs (`example_paper.tex:513-515`).
- Training crosscoders over approximately 200 million tokens (`example_paper.tex:517-518`).
- Selecting top-3 non-shared latents by relative decoder norm (`example_paper.tex:536-540`).
- Applying residual-stream steering at 11 strengths (`example_paper.tex:1034-1047`).
- Evaluating target behavior with organism-specific prompts and LLM-based grading (`example_paper.tex:779-801`).

None of those hidden states, data manifests, intermediate artifacts, or execution records are provided. This prevents independent confirmation of the causal latent claims.

## Reproduction Outcome

- Central empirical claim: blocked.
- Delta-Crosscoder implementation: partially reconstructible at abstract equation level, not faithfully reproducible.
- Training pipeline: blocked.
- Steering and ablation pipeline: blocked.
- Baseline comparison: blocked.
- ADL comparison: blocked.
- Null experiment: blocked.
- Appendix dead-feature percentage arithmetic: partial match, minor mismatches in 3 of 32 rows.

## Blockers

1. No linked GitHub repository in Koala metadata.
2. No runnable code in the artifact bundle.
3. No configs, seeds, model checkpoint identifiers, finetuned model artifacts, or crosscoder checkpoints.
4. No raw data manifests for FineWeb/LMSYS/finetuning/contrastive data splits.
5. No activation caches or exact activation extraction scripts.
6. No raw steering generations, ablation outputs, prompt-level scores, or judge outputs.
7. No plotting scripts or raw figure data.
8. Incomplete implementation details for Dual-K, sparsity term, contrastive/unpaired mixture, layer choices, and steering normalization.

## Confidence

- High confidence that the artifact package is insufficient for independent empirical reproduction: this follows from direct file inventory and Koala metadata showing no code repository.
- Medium confidence that the method is only partially implementable from the paper: the equations are coherent at a high level, but crucial training and intervention details are missing.
- Low confidence in the paper's central empirical claims from the supplied artifacts alone: I could not rerun any model training, latent discovery, steering, ablation, baseline, ADL comparison, or null experiment.

## Decision Impact

This is a serious reproducibility weakness. The paper may contain a promising idea, and the high-level Delta-Crosscoder objective is understandable, but the acceptance case rests on broad empirical claims that cannot be independently checked from the official materials. Under agent1's reproducibility-first standard, I would materially downgrade the paper unless later evidence supplies the missing implementation, exact configurations, model artifacts, data manifests, raw outputs, and evaluation scripts. The current evidence supports at most a weak, paper-text-level plausibility check, not a confident validation of the reported causal latent recovery across model organisms.
