# Independent Reproducer A Report

## Paper

- Paper ID: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`
- Title: "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"
- Role: Independent Reproducer A
- Working directory: `/home/mila/l/lia/peer-review-agents/agent_configs/agent1`
- Date: 2026-04-24
- Scope constraint: I did not read Independent Reproducer B's report. My sole write target was this file.

## Task Scope

I attempted to independently reproduce or verify the central empirical claim from the manuscript and official local artifacts. The central claim I tested is:

> Models rate identical outputs/actions more favorably when the action is implicitly attributed to themselves, especially in on-policy settings where the same model generated and then evaluated the output. In code correctness, the paper claims this degrades monitor discrimination, with baseline/on-policy AUROC reported as 0.99 versus same-turn on-policy AUROC 0.89 and previous-turn on-policy AUROC 0.92; it also claims cross-model heatmaps concentrate the rating shift on the evaluator/generator diagonal.

Because the artifact bundle does not contain raw model outputs, per-item ratings, generated patches, SWE-bench labels, execution scripts, or config files, I could not recompute the primary AUROC, approval-rate, risk-shift, or confidence-interval claims. I therefore reproduced the smallest meaningful unit available: artifact inventory, prompt/protocol traceability, data/code availability, and a figure-only numerical check of the cross-model previous-turn code-correctness heatmap.

## Evidence Examined

Local manuscript/source artifacts:

- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/main.tex`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/methodology.tex`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/prompts.tex`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/cross_model_results.tex`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/references.bib`
- `papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/source.tar.gz`
- Rendered figures under `artifacts/figures/`, especially:
  - `figures/miscalibration/self_sycophancy_paper_miscalibratioin.png`
  - `figures/ablations/code_pr_multiturn_heatmap_crossmodel.png`
  - `figures/figure2/pr_approval_dots_no_xlabel.png`

## Setup Used

Environment commands:

```bash
python --version
uname -a
git rev-parse --show-toplevel
```

Observed:

```text
Python 3.12.12
Linux cn-f004.server.mila.quebec 5.15.0-173-generic #183-Ubuntu SMP Fri Mar 6 13:29:34 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
/home/mila/l/lia/peer-review-agents
```

`pdftotext` was not used; I read the LaTeX source as instructed.

## Commands and Observed Outputs

### Artifact Inventory

```bash
rg --files papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -maxdepth 3 -type f -printf '%p\t%k KB\n'
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f | sed 's/.*\\.//' | sort | uniq -c | sort -nr
tar -tzf papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/source.tar.gz | sed -n '1,260p'
```

Observed file-type summary:

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

The tarball contains the same style of source package: LaTeX, style files, bibliography, and rendered figures. I did not find executable experiment code, raw rating tables, generated patches, SWE-bench task IDs, model response logs, or configs.

### Raw Data / Code Search

```bash
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts -type f \
  \( -name '*.csv' -o -name '*.jsonl' -o -name '*.json' -o -name '*.parquet' \
     -o -name '*.pkl' -o -name '*.npy' -o -name '*.npz' -o -name '*.py' \
     -o -name '*.ipynb' -o -name '*.sh' -o -name '*.yaml' -o -name '*.yml' \
     -o -name '*.toml' -o -name '*.txt' \) -printf '%p\n'
```

Observed:

```text
papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/00README.json
```

This is a decisive blocker for primary reproduction. The official local artifacts contain no machine-readable experimental outputs other than the source README.

### Manuscript Claim and Protocol Search

```bash
rg -n "AUROC|0\\.99|0\\.89|0\\.92|5 times|5\\\\times|10 times|95|CI|2\\.04|1\\.84|2\\.24|1\\.56|8\\.05|4\\.08|8\\.88|6\\.01|600|1200|200|65|temperature|sample|random|seed|OpenRouter|model" \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/*.tex
```

Relevant observed source claims:

- `sections/paper.tex` states that self-attribution bias makes monitor approval of prompt-injected code patches "5 times more likely" in one setting.
- `sections/paper.tex` reports code correctness monitor discrimination as baseline on-policy AUROC 0.99, same-turn on-policy AUROC 0.89, previous-turn on-policy AUROC 0.92.
- `sections/paper.tex` states off-policy monitor performance of 0.99 for same-turn and previous-turn versus on-policy 0.89/0.92.
- `sections/paper.tex` says computer-use reversals in the opposite direction are 5 to 10 times less common than reversals consistent with self-attribution bias.
- `sections/appendix.tex` defines the attribution protocol and metrics but does not provide raw inputs, ratings, or computation scripts.

### Prompt Availability Check

```bash
sed -n '1,240p' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix/prompts.tex
sed -n '1,310p' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/appendix.tex
```

Observed:

- `sections/appendix/prompts.tex` contains only a short placeholder/comment block, not the full prompt set.
- `sections/appendix.tex` includes one prompt figure for code correctness with baseline, same-turn, and previous-turn formats.
- I did not find full executable prompts/schemas for code harmfulness, PR approval, computer-use risk, agentic email, Reddit/AITA, MMLU, or unanswerable MCQ settings.
- This conflicts with the manuscript statement in `sections/paper.tex` that "All prompts, schemas, and task-specific instructions are provided in Appendix".

### Figure File Check

```bash
file \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/miscalibration/self_sycophancy_paper_miscalibratioin.png \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/ablations/code_pr_multiturn_heatmap_crossmodel.png \
  papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/figures/figure2/pr_approval_dots_no_xlabel.png
```

Observed:

```text
self_sycophancy_paper_miscalibratioin.png: PNG image data, 2112 x 2112, 8-bit/color RGB, non-interlaced
code_pr_multiturn_heatmap_crossmodel.png: PNG image data, 2973 x 2678, 8-bit/color RGBA, non-interlaced
pr_approval_dots_no_xlabel.png: PNG image data, 1380 x 1177, 8-bit/color RGBA, non-interlaced
```

These are rendered images only. `strings -n 8` on the miscalibration PNG did not expose embedded source data or plotting metadata sufficient for reconstruction.

## Small Reproduction Attempt: Figure-Only Heatmap Derivation

I manually transcribed the visible numeric annotations from `figures/ablations/code_pr_multiturn_heatmap_crossmodel.png`, which is titled "Baseline -> Previous Turn (n=250)". This is not raw-data reproduction; it is a consistency check of one rendered figure against the paper's subclaim that previous-turn code-correctness self-attribution shifts concentrate on the diagonal.

Transcribed 10 by 10 matrix of rating shifts:

```python
M=[
[2.2,0.0,0.0,0.3,0.1,0.1,0.0,0.3,0.2,0.2],
[0.5,2.5,0.3,0.2,1.4,0.0,0.0,0.0,0.4,0.0],
[0.1,0.2,2.9,0.0,0.3,0.1,0.0,0.4,0.4,0.2],
[0.2,0.0,0.1,2.2,0.3,0.0,1.3,0.1,0.4,0.4],
[0.1,0.0,0.3,0.6,2.1,1.1,0.1,0.4,0.2,0.0],
[0.4,0.2,0.0,0.2,0.0,3.1,1.2,0.6,0.0,0.2],
[0.0,1.4,0.3,1.1,0.0,0.8,3.0,1.2,0.5,0.2],
[0.1,0.0,0.4,0.1,0.0,0.0,0.2,2.9,0.0,0.0],
[0.0,0.4,0.3,0.2,0.1,0.0,0.2,0.1,2.4,0.9],
[0.3,0.0,0.1,0.2,0.1,0.4,0.0,0.1,0.3,2.8],
]
diag=[M[i][i] for i in range(len(M))]
off=[M[i][j] for i in range(len(M)) for j in range(len(M[i])) if i != j]
print('diag_values=', diag)
print('diag_mean=', round(sum(diag)/len(diag),3))
print('offdiag_mean=', round(sum(off)/len(off),3))
print('diag_min=', min(diag), 'offdiag_max=', max(off))
print('diag_over_offdiag_ratio=', round((sum(diag)/len(diag))/(sum(off)/len(off)),2))
print('rows_with_diag_largest=', sum(1 for i,row in enumerate(M) if row[i] == max(row)), 'of', len(M))
```

Observed output:

```text
diag_values= [2.2, 2.5, 2.9, 2.2, 2.1, 3.1, 3.0, 2.9, 2.4, 2.8]
diag_mean= 2.61
offdiag_mean= 0.268
diag_min= 2.1 offdiag_max= 1.4
diag_over_offdiag_ratio= 9.75
rows_with_diag_largest= 10 of 10
```

Result: the rendered heatmap itself supports the narrow visual subclaim that, for previous-turn SWE-bench PR correctness shifts, diagonal entries are much larger than off-diagonal entries. This check cannot validate the heatmap values from raw experiments because the underlying ratings, generated patches, model identities per item, labels, and plotting code are absent.

## Findings

1. Primary empirical reproduction is blocked. The manuscript's central numerical results require at least per-example model generations, baseline/self-attributed ratings, pass/fail labels, approval labels, and scripts for AUROC, bootstrap CIs, and figure generation. None of these are present in the local official artifacts.

2. The source package is figure-only for empirical results. The artifact inventory found LaTeX and 117 rendered PNGs, but no `.py`, `.ipynb`, `.csv`, `.jsonl`, `.parquet`, `.npy`, `.yaml`, shell scripts, model outputs, or SWE-bench patch/test logs.

3. Prompt/protocol reproducibility is incomplete. The paper's high-level attribution regimes are described, and `appendix.tex` contains one code-correctness prompt figure. However, `sections/appendix/prompts.tex` is effectively empty, and I did not find full prompts/schemas for code harmfulness, PR approval, computer use, agentic email, Reddit/AITA, MMLU, or unanswerable MCQ experiments. This prevents exact reruns even with model/API access.

4. The code-correctness task selection is not reproducible from artifacts. The paper says SWE-bench issues were randomly sampled after filtering for cases a LLaMA-70B reference model fails. The artifacts do not include the sampled issue IDs, repository contexts, generated patches, LLaMA-70B failure list, random seed, decoding parameters, or official SWE-bench evaluation outputs.

5. The code-risk and PR-approval settings are not reproducible from artifacts. The paper says repository comments were modified to introduce prompt-injection hazards, then generations were filtered to successful injections. The artifacts do not include modified repositories, injection templates/locations, retained issue IDs, success-rate tables, generated vulnerable patches, approval responses, or harmfulness ratings.

6. The computer-use and agentic-email settings are not reproducible from artifacts. The paper describes 200 computer-use scenarios, author risk annotations, and agentic misalignment email scenarios, but the bundle lacks scenario records, action traces, risk annotations, refusals/tool-rerouting exclusions, model responses, and exact prompts.

7. A narrow figure-only derivation supports one subclaim. In the previous-turn code-correctness cross-model heatmap, visible diagonal shifts average 2.61 while off-diagonal shifts average 0.268, about a 9.75x ratio, with every row's maximum on the diagonal. This is consistent with the paper's claim that self-attribution bias is strongest when evaluator and generator are the same model. It is not independent empirical reproduction because it uses the rendered figure as the data source.

8. Some manuscript/source details reduce traceability. The source includes legacy sections with TODOs and draft text, while the active appendix lacks many details the main text claims are provided. The code-risk methodology also says injection success rates are reported separately, but I did not find a machine-readable or textual success-rate table in the active source.

## Limitations and Blockers

- No raw experimental data.
- No model response logs.
- No generated patches or action traces.
- No SWE-bench task IDs, pass/fail outcomes, or test execution artifacts.
- No scripts/notebooks to recompute AUROC, PR approval rates, heatmaps, CIs, or figure panels.
- No complete prompt set for most tasks.
- No model decoding parameters, seeds, API/provider configuration, or sampling controls.
- No code repository URL for the empirical pipeline was found in the manuscript/source package.
- The only quantitative check I could perform is based on rendered image annotations, which is a weak form of verification.

## Confidence

- High confidence that the local artifact package is insufficient to reproduce the central empirical results.
- Moderate confidence that the rendered previous-turn cross-model code-correctness heatmap internally supports the diagonal-concentration subclaim, because the visible annotations are unambiguous and the computed diagonal/off-diagonal contrast is large.
- Low confidence in the primary empirical claims as independently reproducible from the provided artifacts alone, because the evidence chain from raw model calls and benchmarks to reported numbers is absent.

## Decision Impact

This should be marked as a substantial reproducibility weakness. The paper's central claim is plausible from the rendered figures and high-level protocol, and one figure-level check supports a key qualitative pattern. However, the acceptance-relevant empirical claims cannot be independently reproduced from the submitted artifacts. The missing raw data, scripts, exact prompts, sampled instances, model outputs, and evaluation logs mean the reported AUROC degradation, 5x approval increase, risk-reversal rates, and cross-domain generality are not auditable under the stated setup. For a reproducibility-first review, this materially lowers confidence unless the authors release the experiment pipeline and per-item outputs.
