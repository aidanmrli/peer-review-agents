# Independent Reproducer A Report

Paper: `a0efc92c-0beb-43e5-a52e-ebeb64b90dfc`, "Sequence Diffusion Model for Temporal Link Prediction in Continuous-Time Dynamic Graph"

Role: Independent Reproducer A

Date: 2026-04-24, America/Toronto

## Claim Attempted

I attempted to reproduce the paper's central empirical claim from the submitted paper and official artifacts: SDG "consistently achieves state-of-the-art performance in the temporal link prediction task" (paper source `example_paper.tex`, lines 106-107 and 121-123), with the main evidence being Tables 1 and 2 over 10 temporal link prediction datasets (lines 296-317 and 333-366).

The paper says experiments use DyGLib and TGB-Seq for the common pipelines (lines 378-382), and the appendix gives SDG hyperparameters including neighbor length, diffusion steps, loss weights, batch size, embedding size, and task loss (lines 566-594). The experiment section also states that "The code will be available upon acceptance" (line 328).

## Setup Used

Working directory:

```text
/home/mila/l/lia/peer-review-agents/agent_configs/agent2
```

Artifacts inspected:

```text
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/paper.pdf
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/source.tar.gz
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib
papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq
```

Repository snapshots:

```text
DyGLib HEAD: 3aacc36b94b8d2d8293d70a74fdf6d39089b4163
TGB-Seq HEAD: c1ee801ea4301c2f944f5b3da10cbc5310fa4f68
```

Environment checks:

```text
python --version
# Python 3.12.12

python -c "import torch, numpy, pandas; print('torch', torch.__version__); print('numpy', numpy.__version__); print('pandas', pandas.__version__)"
# ModuleNotFoundError: No module named 'torch'

python -c "import numpy; print('numpy', numpy.__version__)"
# ModuleNotFoundError: No module named 'numpy'
```

`pdftotext` was unavailable, so I used `example_paper.tex` as the authoritative paper text.

## Commands and Evidence

I first checked the artifact structure:

```bash
find papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc -maxdepth 3 -type f -o -type d
ls -la papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts
tar -tzf papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/source.tar.gz | sed -n '1,260p'
```

Observed result: `source.tar.gz` contains only the paper source bundle and figures, not executable SDG code. The available code artifacts are shallow clones of DyGLib and TGB-Seq.

I searched for an SDG/diffusion implementation in the available Python artifacts:

```bash
rg -n --glob '*.py' "SDG|Sequence Diffusion|diffusion|denois|lambda_diff|lambda_inter|Diffusion" \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq
```

Observed result: no matches. The available model files are baseline-only:

```bash
find papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib/models -maxdepth 1 -type f -printf '%f\n' | sort
find papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/examples/models -maxdepth 1 -type f -printf '%f\n' | sort
```

Observed model files in both trees:

```text
CAWN.py
DyGFormer.py
EdgeBank.py
GraphMixer.py
MemoryModel.py
TCL.py
TGAT.py
modules.py
```

I checked whether the documented link-prediction entry points accept `SDG`:

```bash
rg -n "choices|model_name|default='DyGFormer'|JODIE|DyRep|TGN|TGAT|CAWN|TCL|GraphMixer|DyGFormer" \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib/utils/load_configs.py \
  papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/examples/utils/load_configs.py
```

Observed result: both parsers restrict `--model_name` to `['JODIE', 'DyRep', 'TGAT', 'TGN', 'CAWN', 'EdgeBank', 'TCL', 'GraphMixer', 'DyGFormer']`; `SDG` is absent (`DyGLib/utils/load_configs.py`, lines 17-18; `TGB-Seq/examples/utils/load_configs.py`, lines 17-18). The DyGLib training script imports only the baseline models and raises `ValueError(f"Wrong value for model_name {args.model_name}!")` for any other model (`train_link_prediction.py`, lines 13-18 and 97-125).

I then attempted the documented entry-point help commands:

```bash
cd papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib
python train_link_prediction.py --help

cd ../TGB-Seq
python examples/train_link_prediction.py --help
```

Observed result: both fail immediately before argument parsing because `tqdm` is not installed:

```text
ModuleNotFoundError: No module named 'tqdm'
```

Even if dependencies were installed, these entry points still would not run SDG because the parser/model factory do not include SDG.

## Smallest Meaningful Reproduction: Table Arithmetic

Because the SDG implementation and runnable environment were unavailable, I reproduced the smallest meaningful empirical unit available from the paper: the win/loss and improvement arithmetic in Tables 1 and 2.

Command:

```bash
python - <<'PY'
metrics = [
    ('Wikipedia MRR', 89.17, 88.81),
    ('Wikipedia HR@10', 91.40, 90.95),
    ('Reddit MRR', 89.13, 88.95),
    ('Reddit HR@10', 94.76, 94.39),
    ('MOOC MRR', 60.55, 58.79),
    ('MOOC HR@10', 79.93, 78.68),
    ('LastFM MRR', 53.79, 54.53),
    ('LastFM HR@10', 69.15, 69.95),
    ('UCI MRR', 76.13, 75.73),
    ('UCI HR@10', 79.78, 82.39),
    ('GoogleLocal MRR', 62.60, 54.68),
    ('GoogleLocal HR@10', 78.54, 72.24),
    ('YouTube MRR', 60.54, 58.95),
    ('YouTube HR@10', 71.01, 71.61),
    ('Flickr MRR', 61.79, 61.27),
    ('Flickr HR@10', 80.92, 79.11),
    ('ML-20M MRR', 36.63, 36.01),
    ('ML-20M HR@10', 53.55, 52.28),
    ('Taobao MRR', 69.70, 67.41),
    ('Taobao HR@10', 81.43, 79.08),
]
wins = 0
losses = []
for name, sdg, best in metrics:
    abs_imp = sdg - best
    rel_imp = 100.0 * abs_imp / best
    if abs_imp > 0:
        wins += 1
    else:
        losses.append((name, sdg, best, abs_imp, rel_imp))
    print(f'{name}: SDG={sdg:.2f}, best_baseline={best:.2f}, abs={abs_imp:.2f}, rel={rel_imp:.2f}%')
print(f'wins={wins}/{len(metrics)}')
print('losses=')
for row in losses:
    print(f'  {row[0]}: abs={row[3]:.2f}, rel={row[4]:.2f}%')
PY
```

Observed result:

```text
wins=16/20
losses=
  LastFM MRR: abs=-0.74, rel=-1.36%
  LastFM HR@10: abs=-0.80, rel=-1.14%
  UCI HR@10: abs=-2.61, rel=-3.17%
  YouTube HR@10: abs=-0.60, rel=-0.84%
```

This table-level check partially matches the printed table values but contradicts the strongest wording of the claim. SDG is not best on all reported metrics. The paper itself acknowledges LastFM and UCI degradation for seen datasets (line 399), but still claims in the abstract and contributions that SDG consistently achieves/outperforms SOTA (lines 107 and 123). The unseen-dataset discussion also states that "Across all five datasets, SDG consistently achieves the best performance" and that HR@10 improves by `1.59%-8.72%` (line 401), but Table 2 reports YouTube HR@10 as `71.01` for SDG versus `71.61` for TGN, i.e. `-0.60` absolute and `-0.84%` relative (lines 349-365).

I also found one table arithmetic inconsistency: Table 1 reports Wikipedia HR@10 `Abs.Imprv. = 0.63` and `Rel.Imprv. = 0.71%` (lines 316-317), but the printed values are SDG `91.40` versus DyGFormer `90.95` (lines 311 and 314), which gives `0.45` absolute and about `0.49%` relative.

## Observed Result

Full empirical reproduction is blocked. The official artifacts available to me do not include an SDG implementation, an SDG training/evaluation command, run logs, checkpoints, random seeds beyond the high-level "3 runs" statement, processed datasets for the reported experiments, or a dependency-pinned runnable environment. The linked/cloned repositories are DyGLib and TGB-Seq baseline libraries; their training argument parsers do not include SDG.

The smallest table-level reproduction gives a partial mismatch with the central empirical framing:

- SDG wins 16 of 20 reported Table 1/2 metric columns, not all 20.
- SDG loses on LastFM MRR, LastFM HR@10, UCI HR@10, and YouTube HR@10.
- The paper's YouTube HR@10 discussion is inconsistent with its own Table 2.
- The Wikipedia HR@10 improvement row in Table 1 does not match the printed metric values.

## Match Status

Blocked for full reproduction; partial mismatch for the smallest reproducible table-level check.

## Concrete Reason for Failure

The central empirical claim cannot be independently reproduced from the provided artifacts because the submitted code artifacts do not contain SDG. The paper points to code availability only "upon acceptance" (line 328), and the available DyGLib/TGB-Seq clones expose only baseline models. The local runtime also lacks required dependencies (`tqdm`, `numpy`, `torch`), but this is secondary: installing dependencies would still not supply the missing SDG model, diffusion decoder, SDG losses, tuned configurations as executable configs, training scripts, checkpoints, or result logs.

## Reproducibility Outcome

Independent Reproducer A outcome: weak reproducibility for the central empirical claim. I could not reproduce SDG's reported temporal link prediction performance from the artifacts. The most meaningful fallback check, table arithmetic, finds that the reported numbers themselves do not support a literal "consistently best" claim across all metrics.

I did not inspect or copy Independent Reproducer B's reasoning, per the role instruction.

## Score Impact

This should materially reduce confidence in the paper's empirical acceptance case. A method paper claiming consistent SOTA across dynamic graph benchmarks needs executable method code or enough logs/configuration to independently recover the key results. Here, the core method is absent from the artifacts, and the paper's own tables partially contradict the strongest empirical wording.
