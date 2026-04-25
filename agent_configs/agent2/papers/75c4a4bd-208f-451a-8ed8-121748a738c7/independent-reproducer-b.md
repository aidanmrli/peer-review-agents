# Independent Reproducer B Report

Paper: `75c4a4bd-208f-451a-8ed8-121748a738c7`, "Plain Transformers are Surprisingly Powerful Link Predictors"

Role: Independent Reproducer B. I used a table-arithmetic and source-consistency route rather than trying to reconstruct Reproducer A's path. I did not modify any paper artifact or repository state.

## Claim Attempted

I attempted to verify the paper's central empirical claim from the available artifacts:

- PENCIL, a plain BERT-style Transformer over sampled local subgraphs, is a surprisingly strong link predictor without hand-crafted heuristic features or node IDs.
- The reported original and HeaRT benchmark tables support state-of-the-art or near-state-of-the-art performance.
- The reported standard deviations support a stability claim.
- The parameter/time table and ogbl-ppa comparison figure support claims of parameter efficiency and practical efficiency.
- The multiplicative residual ablation and depth figure support claims that the explicit structural residual and deeper Transformer stacks matter.

Primary paper locations checked:

- Abstract claim: `artifacts/main.tex:132`
- ogbl-ppa parameter-efficiency figure caption: `artifacts/main.tex:140-144`
- contribution claim of `22x` to `146x` fewer learnable parameters and `6.7x` to `40x` fewer epochs: `artifacts/main.tex:151-153`
- original benchmark table: `artifacts/main.tex:331-368`
- HeaRT benchmark table: `artifacts/main.tex:374-404`
- main-results interpretation: `artifacts/main.tex:413-418`
- depth interpretation: `artifacts/main.tex:420-423`
- implementation and heuristic-regression setup: `artifacts/main.tex:776-808`
- original and HeaRT hyperparameter tables: `artifacts/main.tex:849-891`
- multiplicative residual ablation: `artifacts/main.tex:967-984`
- computational/time table: `artifacts/main.tex:993-1034`

## Independent Route Used

I did not attempt a full training run because the provided local artifacts contain only LaTeX source, a PDF, bibliography, style files, and static figures. The source tarball contains the same paper source files and no implementation, raw logs, per-seed metrics, checkpoints, configuration files, or dataset preprocessing scripts.

I used an independent numerical audit:

1. Read `skills/independent-reproducer-b.md`.
2. Inspected `artifacts/main.tex`, `artifacts/source.tar.gz`, and the static figures under `artifacts/figures/`.
3. Queried Koala metadata for the paper. It reports `github_repo_url: null` and `github_urls: []`, so no official code repository is linked through the platform.
4. Transcribed the relevant table values into a small Python script and recomputed ranks, ablation deltas, standard-deviation ranks, parameter ratios, and time ratios.
5. Visually inspected the two figure-only claims: `figures/ogbl_ppa_comparison.png` and `figures/combined_layers_vs_hk.png`. The PNGs contain only Matplotlib metadata, not embedded numeric source data.

Environment:

- Working directory: `/home/mila/l/lia/peer-review-agents/agent_configs/agent2`
- Timestamp: `2026-04-24T22:00:09-04:00`
- Python: `Python 3.12.12`
- No external future-impact or venue-outcome sources were used.

Representative commands:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,240p' skills/independent-reproducer-b.md
rg --files papers/75c4a4bd-208f-451a-8ed8-121748a738c7
tar -tzf papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz | sed -n '1,120p'
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '331,418p'
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '849,895p'
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '967,1035p'
python - <<'PY'
# table values were manually transcribed from main.tex, then sorted by metric
# to recompute top-3 ranks, PENCIL ranks, ablation gains, parameter ratios,
# time ratios, and epoch-ratio sanity checks.
PY
```

## Observed Results

### Artifact Completeness

The artifact set is insufficient for a true empirical reproduction. There is no runnable PENCIL implementation, dependency file, seed list, exact optimizer/scheduler specification, raw per-seed outputs, trained checkpoints, evaluator invocation, or dataset preprocessing script. The paper states that it uses ShaDowKHop/GraphGPT sampling, Hugging Face BERT, PyTorch, and PyG (`main.tex:776`), but those statements do not define an executable reproduction.

The source tarball only contains:

- `main.tex`, `main.bib`, style files
- paper figures
- `paper.pdf`
- no code, logs, configs, checkpoints, or processed datasets

Koala paper metadata confirms no official GitHub link: `github_repo_url` is `null` and `github_urls` is empty.

Outcome for full empirical reproduction: **blocked / weak reproducibility**.

### Original Benchmark Table Ranks

I recomputed ranks from the complete original table, including the pure heuristic rows shown in the appendix. Higher is better for all listed metrics.

Original setting top-3 and PENCIL-family ranks:

| Dataset | Top result from table | PENCIL rank | PENCIL w/o Features rank |
|---|---:|---:|---:|
| cora | PENCIL w/o Features 42.23 | 7th, 32.12 | 1st, 42.23 |
| citeseer | LPFormer 65.42 | 9th, 43.74 | 8th, 47.51 |
| pubmed | NBFNet 44.73 | 3rd, 38.34 | 4th, 38.28 |
| ogbl-collab | LPFormer 68.14 | 4th, 66.56 | 3rd, 66.88 |
| ogbl-ppa | PENCIL 79.54 | 1st, 79.54 | 3rd, 73.85 |
| ogbl-citation2 | MPLP+ 90.72 | 8th, 86.86 | 9th, 86.74 |

Interpretation:

- The text claim that PENCIL achieves SOTA on `ogbl-ppa` is reproduced from the table.
- The text claim that PENCIL achieves SOTA on `cora` is only reproduced if "PENCIL" includes the `PENCIL w/o Features` variant. The strict `PENCIL (Ours)` row is 7th on `cora`.
- The broad abstract claim that PENCIL "outperforms heuristic-informed GNNs" is not generally supported by the table. LPFormer, NCN/NCNC, and other baselines beat PENCIL-family rows on several datasets.

### HeaRT Benchmark Table Ranks

HeaRT setting top-3 and PENCIL-family ranks:

| Dataset | Top result from table | PENCIL rank | PENCIL w/o Features rank |
|---|---:|---:|---:|
| cora | LPFormer 16.80 | 10th, 13.13 | 6th, 14.63 |
| citeseer | NCN 28.65 | 8th, 16.80 | 9th, 16.50 |
| pubmed | LPFormer 9.99 | 3rd, 8.88 | 8th, 7.05 |
| ogbl-collab | LPFormer 7.62 | 8th, 5.40 | 9th, 5.25 |
| ogbl-ppa | PENCIL 45.43 | 1st, 45.43 | 2nd, 44.57 |
| ogbl-ddi | PENCIL w/o Features 14.07 | N/A | 1st, 14.07 |
| ogbl-citation2 | LPFormer 24.70 | 2nd, 23.43 | 3rd, 23.36 |

Interpretation:

- The HeaRT `ogbl-ppa` top-score claim is reproduced from the table.
- The HeaRT `ogbl-ddi` top-score claim is only reproduced for `PENCIL w/o Features`; strict `PENCIL (Ours)` is N/A.
- The HeaRT table does not support a broad dominance claim. PENCIL-family rows are weak on `cora`, `citeseer`, and `ogbl-collab`.

### Stability Claims

The main text claims that PENCIL "consistently" has lower standard deviations than baselines, especially on `ogbl-ppa` with `+/- 0.07` (`main.tex:416`).

Recomputed standard-deviation ranks for strict `PENCIL (Ours)`:

| Setting | Dataset | PENCIL std rank |
|---|---|---:|
| original | cora | 3rd |
| original | citeseer | 6th |
| original | pubmed | 9th |
| original | ogbl-collab | 1st |
| original | ogbl-ppa | 1st |
| original | ogbl-citation2 | 8th |
| HeaRT | cora | 5th |
| HeaRT | citeseer | 10th |
| HeaRT | pubmed | 6th |
| HeaRT | ogbl-collab | 2nd |
| HeaRT | ogbl-ppa | 6th |
| HeaRT | ogbl-citation2 | 3rd |

Interpretation:

- The original `ogbl-ppa` low-variance fact is reproduced: strict PENCIL has `+/- 0.07`, the lowest standard deviation in that column.
- The "consistently lower standard deviations" claim is contradicted by the tables. PENCIL is not lowest on most columns, and on HeaRT `ogbl-ppa`, `PENCIL w/o Features`, LPFormer, NCN, NCNC, and SAGE all have smaller listed standard deviations than strict PENCIL.
- If the authors literally mean variance rather than standard deviation on original `ogbl-ppa`, the variance ratio between the closest PENCIL-family competitor and strict PENCIL is `(0.40 / 0.07)^2 = 32.65`. This supports "much smaller" variance for that one column, but it does not support the global consistency claim.

### Parameter Efficiency and Time Table

The `22x` to `146x` parameter-efficiency claim is not fully reproducible from the paper source because exact parameter counts for all points in `figures/ogbl_ppa_comparison.png` are not tabulated. The static figure visually places PENCIL near the low tens of millions of parameters and several ID-based methods between roughly tens/hundreds and thousands of millions, but that is not enough to independently verify the exact `22x` and `146x` endpoints.

The explicit parameter/time table gives a different local comparison against GAT:

| Model pair | Parameter ratio |
|---|---:|
| PENCIL-3L / GAT-3L | 10.2M / 1.32M = 7.73x more PENCIL parameters |
| PENCIL-8L / GAT-8L | 27.3M / 3.95M = 6.91x more PENCIL parameters |

This exactly supports the appendix statement that PENCIL has roughly `7x` to `8x` more parameters than GAT in those timing settings (`main.tex:1001`). It does not support a general claim that PENCIL is parameter-efficient relative to non-ID GNN baselines.

Time ratios from Table `tab:training-inference-time`:

| Dataset/task | 3L PENCIL/GAT | 8L PENCIL/GAT |
|---|---:|---:|
| ogbl-collab training | 0.019 / 0.021 = 0.90x |
| ogbl-collab inference | 0.012 / 0.015 = 0.80x |
| ogbl-collab training | 0.035 / 0.032 = 1.09x |
| ogbl-collab inference | 0.020 / 0.017 = 1.18x |
| pubmed training | 0.016 / 0.011 = 1.45x |
| pubmed inference | 0.010 / 0.007 = 1.43x |
| pubmed training | 0.029 / 0.020 = 1.45x |
| pubmed inference | 0.016 / 0.011 = 1.45x |

Interpretation:

- The claim that GAT is consistently faster on `pubmed` is reproduced.
- The claim that costs are comparable on `ogbl-collab` is partially reproduced; PENCIL is faster for 3L but slower for 8L.
- The stronger framing that PENCIL is "extremely efficient" is not independently established from the table alone. The listed standard deviations are larger than the means in several entries, so these timing comparisons are noisy.

### Epoch Efficiency

The PENCIL epoch counts in the hyperparameter tables reproduce the cited values:

- `ogbl-citation2`: 0.5 epochs
- `ogbl-ddi`: 8 epochs
- `ogbl-ppa`: 15 epochs

However, the claimed `6.7x` to `40x` fewer epochs than pure GNN architectures is not directly reproducible from the local artifacts because the pure-GNN epoch counts are not tabulated beside PENCIL. Using only the paper's prose range of `20-100+` epochs, possible ratios vary widely:

- `ogbl-citation2`: 20 / 0.5 = 40x and 100 / 0.5 = 200x
- `ogbl-ddi`: 20 / 8 = 2.5x and 100 / 8 = 12.5x
- `ogbl-ppa`: 20 / 15 = 1.33x and 100 / 15 = 6.67x

The exact `6.7x` to `40x` range therefore depends on unstated dataset-specific pure-GNN references. I could not reproduce it from the provided artifact set.

### Multiplicative Residual Ablation

I recomputed every "Performance Gain" entry in Table `tab:multplicative-residual`:

```text
[42.23 - 34.64, 43.34 - 35.98, 67.32 - 58.67,
 38.28 - 21.49, 43.61 - 21.81, 57.49 - 31.81,
 53.47 - 49.52, 66.88 - 53.43, 69.75 - 56.04]
= [7.59, 7.36, 8.65, 16.79, 21.80, 25.68, 3.95, 13.45, 13.71]
```

The table arithmetic is correct. This supports the narrower claim that the multiplicative residual substantially improves the reported metrics in the three listed settings. It also weakens any interpretation that the input encoding alone is sufficient.

### Depth Claim

The depth claim at `main.tex:420-423` is qualitative and figure-derived. Visual inspection of `figures/combined_layers_vs_hk.png` supports the specific statement that H@50 and H@100 improve with more layers across `cora`, `pubmed`, and `ogbl-collab`. The same figure does not show monotonic improvement for H@20: `cora` H@20 dips at 8 layers and `ogbl-collab` H@20 peaks around 2 layers and then declines.

Because no raw depth-sweep CSV/log values are present, this remains a visual source-consistency check, not an empirical reproduction.

## Match, Partial Match, Mismatch, or Blocked

Overall outcome: **blocked for full empirical reproduction; partial match for static table arithmetic; contradicted for several broad textual generalizations.**

Matched from the paper source:

- Original `ogbl-ppa` top rank for strict PENCIL.
- Original `cora` top rank for the PENCIL-family row `PENCIL w/o Features`.
- HeaRT `ogbl-ppa` top rank for strict PENCIL.
- HeaRT `ogbl-ddi` top rank for `PENCIL w/o Features`.
- Multiplicative residual gain arithmetic.
- PENCIL-vs-GAT parameter ratios and timing ratios in the appendix table.
- Qualitative H@50/H@100 depth trend from the static figure.

Mismatched or overclaimed:

- "PENCIL outperforms heuristic-informed GNNs" is not generally supported across the original or HeaRT tables.
- "PENCIL achieves SOTA on cora" is only true if the featureless variant is included under the same name.
- "PENCIL secures top scores on ogbl-ddi" is only true for `PENCIL w/o Features`; strict `PENCIL (Ours)` is N/A.
- "Consistently lower standard deviations" is contradicted by the rank recomputation.
- Exact `22x` to `146x` parameter savings and exact `6.7x` to `40x` epoch savings are not reproducible from the provided artifacts.

Blocked:

- Regenerating Table 1/Table 2 means and standard deviations from seeds.
- Running the HeaRT or original benchmark evaluation.
- Verifying the checkpoint-selection rule for the `ogbl-ppa` HeaRT result, which the paper says reuses the optimal original-setting checkpoint (`main.tex:849`).
- Verifying the parameter-efficiency figure's exact x-axis values.
- Verifying the initialization and batching figures numerically.

## Agreement With Reproducer A

After completing the independent table/rank calculations above, I checked Reproducer A only for a final agreement note. My outcome agrees with Reproducer A's high-level conclusion: the central empirical claim is weakly reproducible from the supplied artifacts because there is no runnable implementation, no official GitHub link, no logs, no checkpoints, and no raw per-seed outputs. My independent contribution is narrower and arithmetic-focused: several table-derived claims are internally consistent, but the broad SOTA, stability, parameter-efficiency, and epoch-efficiency prose is materially overgeneralized or not reproducible from the artifacts.

## Final Synthesis and Score Impact

The paper's static tables support a limited version of the acceptance case: PENCIL-family variants are genuinely strong on `ogbl-ppa`, `ogbl-ddi` under HeaRT, and a few near-top settings; the multiplicative residual ablation arithmetic is correct; and the depth figure visually supports H@50/H@100 gains.

The central claim is not independently reproducible in the empirical sense. The available artifacts let me audit table arithmetic, not regenerate the results. Several headline interpretations are overstated relative to the tables, especially global superiority over heuristic-informed baselines and consistent stability. I would mark reproducibility as **weak** and score-impact as materially negative, with the main caveat that the reported numbers may still be correct but cannot be verified from the provided materials.
