# Independent Reproducer B Report

Paper: `a0efc92c-0beb-43e5-a52e-ebeb64b90dfc`, "Sequence Diffusion Model for Temporal Link Prediction in Continuous-Time Dynamic Graph"

Role: Independent Reproducer B

## Claim Attempted

I attempted to validate the paper's central empirical claim through a metric-definition and table-consistency route: SDG is claimed to "consistently" achieve state-of-the-art temporal link prediction performance (paper source `artifacts/example_paper.tex`, lines 105-108 and 119-123), with the main evidence in Tables 1-3 and the discussion around lines 399-407 and 619-656.

The specific claim tested here is not whether a full training run can be repeated, but whether the metric code and reported tables support the stated SOTA conclusion and whether the supplied artifacts contain enough executable evaluation logic to trace those reported metrics.

## Independent Route Used

I used a different route from a direct training rerun:

- Static trace of the supplied TGB-Seq and DyGLib metric/evaluation code.
- Search for an executable SDG path in the supplied artifacts.
- Manual and pure-Python recomputation of ranks and improvements from the paper's printed table values.
- Post hoc comparison with Independent Reproducer A only after my independent conclusion was formed.

## Artifacts and Environment

Working directory:

```text
/home/mila/l/lia/peer-review-agents/agent_configs/agent2
```

Artifacts inspected:

```text
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/source.tar.gz
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq
```

Repository snapshots:

```text
DyGLib HEAD: 3aacc36b94b8d2d8293d70a74fdf6d39089b4163
TGB-Seq HEAD: c1ee801ea4301c2f944f5b3da10cbc5310fa4f68
```

The surrounding worktree already had unrelated dirty changes outside `agent2`; I did not modify them.

## Commands, Calculations, and Trace Steps

I first checked the paper source bundle:

```bash
tar -tzf papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/source.tar.gz
cat papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/00README.json
```

Observed: `source.tar.gz` contains the LaTeX source and figures only. It does not contain SDG training code, result logs, checkpoints, or executable experiment configs. The paper itself states, "The code will be available upon acceptance" (`example_paper.tex`, line 328).

I searched the supplied code artifacts for SDG/diffusion implementation terms:

```bash
rg -n --glob '*.py' --glob '*.md' --glob '*.json' \
  "SDG|Sequence Diffusion|Diffusion|CrossTransformer|denoise|lambda_diff|lambda_inter|diffusion" \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq
```

Observed: no Python/Markdown/JSON matches for SDG or the diffusion method. The TGB-Seq training parser restricts `--model_name` to baseline methods only (`examples/utils/load_configs.py`, lines 17-18), and the model factory raises on any unsupported model (`examples/train_link_prediction.py`, lines 115-145). Thus the supplied executable code cannot instantiate SDG.

I traced metric definitions:

```bash
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/tgb_seq/LinkPred/evaluator.py
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/examples/evaluate_models_utils_mrr.py | sed -n '1,190p'
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib/utils/metrics.py
rg -n "HR@10|hit|hits|Hit|hr@|ranking_list|mrr_list|return \\{'mrr'" \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib \
  --glob '*.py' --glob '*.md'
```

Observed:

- TGB-Seq's evaluator computes MRR by comparing the positive score to each negative score with optimistic/pessimistic tie averaging (`tgb_seq/LinkPred/evaluator.py`, lines 22-28).
- `examples/evaluate_models_utils_mrr.py` consumes predefined negative samples when present and returns only `{'mrr': np.mean(evaluate_metrics)}` (`lines 47-48`, `165-170`).
- DyGLib's `utils/metrics.py` computes AP and ROC-AUC only (`lines 5-19`).
- I found no supplied implementation of HR@10 or hits@K in the cloned Python/Markdown files, even though Tables 1 and 2 report HR@10. Therefore the HR@10 results are not traceable through the provided evaluation artifacts.

I attempted to import the supplied evaluator for a tiny metric sanity check:

```bash
python - <<'PY'
import importlib.util
from pathlib import Path
import numpy as np
p = Path('papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/tgb_seq/LinkPred/evaluator.py')
spec = importlib.util.spec_from_file_location('evalmod', p)
mod = importlib.util.module_from_spec(spec)
spec.loader.exec_module(mod)
PY
```

Observed: `ModuleNotFoundError: No module named 'numpy'`. This local environment is not dependency-complete. I therefore reproduced the evaluator formula manually in pure Python:

```bash
python - <<'PY'
def tgb_mrr_list(pos, negs):
    out = []
    for p, row in zip(pos, negs):
        optimistic = sum(n > p for n in row)
        pessimistic = sum(n >= p for n in row)
        rank = 0.5 * (optimistic + pessimistic) + 1
        out.append(1.0 / rank)
    return out
print('toy_mrr_list', [round(x, 6) for x in tgb_mrr_list([0.5,0.5,0.5], [[0.4,0.3],[0.6,0.4],[0.5,0.6]])])
PY
```

Observed:

```text
toy_mrr_list [1.0, 0.5, 0.4]
```

This matches the static evaluator logic: a positive above all negatives has reciprocal rank 1, one negative above it gives 1/2, and one tie plus one higher negative gives average rank 2.5 and reciprocal rank 0.4.

I then recomputed the table-derived conclusions from the printed values in Tables 1 and 2 (`example_paper.tex`, lines 296-366). The pure-Python recomputation reported:

```text
seen summary {'best': 7, 'second': 3, 'not_best': 3,
 'negative_cells': [('LastFM MRR', -0.74, -1.36, 'CRAFT'),
                    ('LastFM HR', -0.8, -1.14, 'CRAFT'),
                    ('UCI HR', -2.61, -3.17, 'DyGFormer')]}

unseen summary {'best': 9, 'second': 1, 'not_best': 1,
 'negative_cells': [('YouTube HR', -0.6, -0.84, 'TGN')]}
```

Thus SDG is best in 16/20 main reported metric columns, not all 20.

I also recomputed the point-wise AP/AUC average ranks in Table 3 (`example_paper.tex`, lines 619-656). Using the printed values, SDG's ranks are `[2, 3, 1, 1, 2, 2, 1, 1]`, giving an average rank of `1.625`, not the reported `1.50`. The discrepancy comes from Wikipedia ROC-AUC: Table 3 marks SDG `98.23` as second, but TGN has `98.37`, so SDG is third behind DyGFormer and TGN (`example_paper.tex`, lines 635-636).

Finally, I checked the ablation table and narrative (`example_paper.tex`, lines 405-424). The text says removing any component "consistently degrades performance," but the MLP ablation is better than SDG on Wikipedia MRR and HR@10: `89.40/91.69` for MLP versus `89.16/91.45` for SDG (`lines 417-418`). That weakens the claim that every listed design choice contributes consistently.

## Observed Result

The metric-code and table route partially supports the paper's weaker claim that SDG is often the strongest method, especially on TGB-Seq MRR. It does not support the literal "consistently achieves state-of-the-art" claim:

- SDG is not best on LastFM MRR, LastFM HR@10, UCI HR@10, or YouTube HR@10.
- The paper's unseen-dataset discussion says HR@10 improves by `1.59%-8.72%` (`line 401`), but YouTube HR@10 is negative: SDG `71.01` versus TGN `71.61` (`lines 349-365`).
- The paper reports HR@10, but the supplied TGB-Seq evaluator exposes MRR only, and the supplied DyGLib metric utility exposes AP/AUC only. I could not trace HR@10 to released code.
- Table 3's AP/AUC average-rank row is not reproduced by the printed table values.
- The ablation narrative overstates consistency because the MLP ablation beats SDG on Wikipedia.

Full empirical reproduction is blocked because the actual SDG implementation, executable configs, checkpoints, result logs, and dependency-complete environment are absent from the supplied artifacts.

## Match Status

Blocked for full empirical reproduction; partial mismatch for the metric/table-derived central claim.

The result is not a full contradiction of SDG's reported numbers. It is a contradiction of the strongest wording: the printed values do not establish consistent SOTA across all reported metrics, and the artifacts do not permit independent verification of the method that produced the values.

## Agreement With Reproducer A

After completing the independent metric/code trace, I read `independent-reproducer-a.md`. My conclusion agrees materially with Reproducer A: the core SDG empirical claim has weak reproducibility because SDG code is absent and the table arithmetic does not support the literal "consistently best" wording.

My additional independent findings are:

- The supplied TGB-Seq evaluator returns MRR only; HR@10 is not traceable in the available code.
- Table 3's AP/AUC average-rank row is inconsistent with its own printed values.
- The ablation narrative is contradicted by the MLP variant outperforming SDG on Wikipedia.

I do not have a substantive divergence from Reproducer A.

## Reproducibility Outcome

Weak reproducibility. Neither a full SDG rerun nor a complete metric trace is possible from the artifacts. The central empirical case can only be checked at the table-arithmetic level, and that check shows that the paper's strongest conclusions are overstated.

## Score Impact

This should materially lower confidence in the paper's acceptance case. A claimed SOTA method on temporal link prediction needs executable method code and traceable metric implementations for all reported metrics. Here, the main method is absent, HR@10 is not traceable in the supplied evaluation code, and multiple paper-level table/narrative inconsistencies weaken the evidential basis for the central claim.
