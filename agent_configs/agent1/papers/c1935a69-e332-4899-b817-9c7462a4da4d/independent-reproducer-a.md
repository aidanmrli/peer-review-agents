# Independent Reproducer A Report

Paper: `c1935a69-e332-4899-b817-9c7462a4da4d`  
Title: "Consensus is Not Verification: Why Crowd Wisdom Strategies Fail for LLM Truthfulness"  
Role: Independent Reproducer A  
Date: 2026-04-24

## Claim Attempted

I attempted to independently reproduce the smallest meaningful unit of the central empirical claim from the official paper/source artifacts:

> Polling-style aggregation does not consistently improve truthfulness over single-sample baselines, even with substantially more samples.

Because the Koala metadata says there is no linked GitHub repository, and the source artifact contains only LaTeX, figures, bibliography/style files, and the PDF, I could not rerun model inference or recompute bootstrap confidence intervals from raw model responses. I therefore reproduced two source-auditable units:

1. Table-derived comparisons in Appendix "Single-Model Polling Results" between `Individual Avg.` and five aggregation rules at `T=1.0`.
2. Benchmark/sample-count arithmetic from the LaTeX sampling protocol.

## Setup Used

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Relevant artifacts:

```bash
papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/00README.json
papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/paper.pdf
```

The source README reports `main.tex` as the toplevel source, with `pdflatex` and TeX Live 2025. I did not read Independent Reproducer B's report.

## Commands and Derivation Steps

I first inspected the role instructions:

```bash
sed -n '1,240p' skills/independent-reproducer-a.md
```

I then searched the paper source for aggregation, polling, table, sample-count, and benchmark claims:

```bash
rg -n "poll|majority|consensus|single|baseline|truth|accuracy|sample|Table|tabular|includegraphics|Figure|HLE|BoolQ|Com2Sense|Future|Predict" \
  papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
```

I extracted the appendix benchmark and per-model result tables:

```bash
sed -n '815,945p' \
  papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
```

The table-derived arithmetic was computed from the visible LaTeX values in Tables `tab:per_model_hle_boolq` and `tab:per_model_future_com2sense`:

```bash
python - <<'PY'
from statistics import mean
rows = [
('HLE','Gemma-3-4B',27.3,28.7,25.8,28.6,37.4,28.7),
('HLE','GPT-OSS-20B',10.1,11.7,12.0,9.2,8.5,11.2),
('HLE','GPT-OSS-120B',11.7,8.7,2.8,8.4,5.6,8.4),
('HLE','Qwen3-32B',28.2,25.1,22.9,17.3,19.6,22.8),
('HLE','Qwen3-235B',21.4,20.2,14.6,14.6,22.8,25.4),
('BoolQ','Gemma-3-4B',61.1,61.0,62.2,62.1,69.9,60.1),
('BoolQ','GPT-OSS-20B',74.0,79.9,74.4,78.0,66.9,80.0),
('BoolQ','GPT-OSS-120B',80.1,80.0,79.9,84.0,80.8,82.0),
('BoolQ','Qwen3-32B',74.8,77.0,74.0,73.8,75.2,75.9),
('BoolQ','Qwen3-235B',69.9,69.9,68.7,71.2,66.9,72.1),
('Predict-the-Future','Gemma-3-4B',52.6,53.8,53.0,52.0,53.1,52.1),
('Predict-the-Future','GPT-OSS-20B',50.1,49.3,51.4,51.8,50.7,47.7),
('Predict-the-Future','GPT-OSS-120B',49.3,45.4,54.0,46.9,51.8,50.4),
('Predict-the-Future','Qwen3-32B',49.8,50.0,48.8,48.1,41.0,50.1),
('Predict-the-Future','Qwen3-235B',51.8,52.1,55.2,53.8,48.9,52.0),
('Com2Sense','GPT-OSS-20B',62.5,38.9,40.2,36.9,38.0,45.0),
('Com2Sense','GPT-OSS-120B',65.0,41.2,38.2,41.1,39.9,45.8),
('Com2Sense','Qwen3-32B',51.7,28.3,40.9,34.0,30.0,32.8),
('Com2Sense','Qwen3-235B',75.6,77.1,72.9,72.2,69.0,75.3),
]
methods = ['Direct Majority','Highest Conf','Conf Weighted','Pred Weighted','Surp. Popular']
print('Rows:', len(rows))
for i,m in enumerate(methods, start=3):
    deltas=[r[i]-r[2] for r in rows]
    wins=sum(d>0 for d in deltas); ties=sum(abs(d)<1e-9 for d in deltas); losses=sum(d<0 for d in deltas)
    print(f'{m}: wins={wins}, ties={ties}, losses={losses}, mean_delta={mean(deltas):+.2f} pp, min={min(deltas):+.1f}, max={max(deltas):+.1f}')
print('q-model pairs with Com2Sense Gemma omitted:', 35*5 + 100*5 + 100*5 + 100*4)
print('responses if 25 samples * 2 temps * 2 experiment types * 2 prompt roles:', (35*5 + 100*5 + 100*5 + 100*4)*25*2*2*2)
print('responses if Com2Sense had 5 models:', (35+100+100+100)*5*25*2*2*2)
print('q-model pairs implied by 375000 / 200:', 375000/200)
flips = [(3.8,500),(3.2,500),(2.3,175),(1.5,400)]
print('weighted flip percent using rounded row rates:', sum(rate*n for rate,n in flips)/sum(n for rate,n in flips))
print('flip denominator:', sum(n for rate,n in flips))
PY
```

## Observed Results

### Table-derived aggregation comparisons

From the 19 per-model/dataset rows reported at `T=1.0`:

| Aggregation rule | Wins vs `Individual Avg.` | Ties | Losses | Mean delta |
|---|---:|---:|---:|---:|
| Direct Majority | 8 | 1 | 10 | -3.62 pp |
| Highest Conf | 7 | 0 | 12 | -3.95 pp |
| Conf Weighted | 7 | 0 | 12 | -4.37 pp |
| Pred Weighted | 8 | 0 | 11 | -4.79 pp |
| Surp. Popular | 10 | 0 | 9 | -2.59 pp |

This table-derived reconstruction supports the narrow claim that no listed aggregation method consistently improves over the single-sample average. In particular, every aggregation rule has a negative mean delta across the extracted rows, and every rule loses on at least 9 of 19 rows.

The benchmark pattern is heterogeneous rather than uniformly negative:

- HLE: mixed; several aggregation improvements exist, including `Pred Weighted` for Gemma-3-4B at +10.1 pp, but other models degrade.
- BoolQ: mostly small to moderate improvements for at least one aggregation rule, though not a single method dominates every model.
- Predict-the-Future: near chance overall; some methods improve by a few points, but the values remain around 50%.
- Com2Sense: aggregation is strongly worse for three of four reported models; only Qwen3-235B direct majority improves slightly (+1.5 pp).

Therefore the table values are consistent with "not consistently improves," but they do not by themselves establish the full paper claim because the raw model responses, bootstrap procedure, and figure-generation scripts are absent.

### Sample-count arithmetic

The appendix states:

- HLE: 35 questions, 5 models -> 175 question-model pairs.
- BoolQ: 100 questions, 5 models -> 500 question-model pairs.
- Predict-the-Future: 100 questions, 5 models -> 500 question-model pairs.
- Com2Sense: 100 questions, Gemma omitted, 4 models -> 400 question-model pairs.

This totals `1,575` question-model pairs, matching the denominator in the temperature-flip table:

```text
175 + 500 + 500 + 400 = 1,575
```

The sampling protocol says `25` samples at each of `2` temperatures for each of `2` experiment types, and each experiment issues both direct-answer and prediction/confidence prompts. Interpreting that as `25 * 2 * 2 * 2 = 200` responses per question-model pair gives:

```text
1,575 * 200 = 315,000 responses
```

If Com2Sense had used all five models despite the table note saying "Com2Sense omits Gemma-3-4B," the count would be:

```text
(35 + 100 + 100 + 100) * 5 * 200 = 335,000 responses
```

Neither calculation reproduces the paper's stated `375,000` total responses. The stated total would imply:

```text
375,000 / 200 = 1,875 question-model pairs
```

That is 300 more question-model pairs than the listed benchmark/model counts with Gemma omitted on Com2Sense, and 200 more than the all-five-model interpretation. I therefore mark the total-response arithmetic as a mismatch unless there is an unreported benchmark, prompt class, model inclusion, or counting convention not present in the LaTeX source.

### Temperature flip arithmetic

The reported benchmark denominators in Table `tab:temp_stability` sum to:

```text
500 + 500 + 175 + 400 = 1,575
```

The weighted average of the rounded row flip rates is:

```text
(3.8*500 + 3.2*500 + 2.3*175 + 1.5*400) / 1575 = 2.8587%
```

This rounds to the reported overall `2.9%`. This source-level arithmetic is internally consistent.

## Match / Partial Match / Mismatch / Blocked

Result: **partial match with a material reproducibility blocker and one arithmetic mismatch.**

What matched:

- The appendix table values support the smallest checked unit of the core empirical claim: across the reported `T=1.0` per-model rows, aggregation does not consistently improve over `Individual Avg.`.
- The temperature-flip denominator and rounded weighted average reproduce the reported overall `2.9%` result from the displayed table.

What did not match:

- The stated `375,000` total responses does not follow from the benchmark counts, model inclusion notes, and sampling protocol in the source. The source-implied count is `315,000` under the table's Com2Sense model omission, or `335,000` if all five models were used everywhere.

What was blocked:

- I could not independently recompute accuracies, confidence intervals, majority votes, confidence-weighted votes, predicted-popularity votes, or Surprisingly Popular outputs from raw responses because no response data, scripts, seeds, environment files, or linked repository are included in the available Koala artifacts.
- The figure PNGs are included, but without the data and plotting code they only allow visual inspection, not independent numerical reproduction.

## Concrete Reason for Failure or Limitation

The official artifacts are sufficient to audit some LaTeX-derived arithmetic and table comparisons, but insufficient to reproduce the empirical pipeline. The core claim depends on generated LLM samples, parsing rules, aggregation implementations, and bootstrap resampling. The LaTeX source gives prompt descriptions and reported results, but does not provide the raw sampled outputs or code needed to verify that the reported tables follow from the model generations.

## Final Synthesis and Score Impact

My independent check gives partial support to the paper's central claim at the level of reported appendix tables: aggregation methods are not consistent winners over the individual-sample baseline, and several are worse on average. However, this is a table audit, not an independent experimental reproduction. The absence of raw model responses and code is a serious reproducibility limitation, and the stated `375,000` response total appears inconsistent with the paper's own benchmark/model/sample counts. I would materially downgrade confidence in the exact empirical accounting and confidence intervals, while retaining moderate confidence in the qualitative "no consistent improvement" conclusion as far as the reported tables themselves show it.
