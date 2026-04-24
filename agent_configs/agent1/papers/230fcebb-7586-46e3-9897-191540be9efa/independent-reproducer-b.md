# Independent Reproducer B Report

## Paper and Role

- Paper ID: `230fcebb-7586-46e3-9897-191540be9efa`
- Title: "Why Depth Matters in Parallelizable Sequence Models: A Lie Algebraic View"
- Assigned role: Independent Reproducer B
- Date: 2026-04-24
- Repo inspected: `papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking`
- Repo HEAD: `e6575fae9d3aa8f32cf30254269054fbf58c87b1`
- Independent route: static code-to-paper trace of the released repository, data generation path, metric logging path, and figure/table recovery path. I did not use Reproducer A's notes; no Reproducer A report was present when checked.

## Claim Attempted

I attempted to verify whether an independent reviewer can recover the paper's reported Table 1 and Figures 2/3, especially the empirical claim that increasing model depth improves maximum accurately tracked sequence length or reduces rotation-prediction MSE.

The relevant paper claims are:

- Table 1: word-problem length generalization accuracy, with models trained on length 128 and tested on length 256, using 500K training sequences except `C_2` with 100K.
- Figure 2: maximum sequence length achieving `>90%` training sequence-level accuracy on `A_5`, with depth varied and models trained on length up to 128.
- Figure 3 and Appendix Figure A.1: per-sequence-length MSE for the `A_5` rotation prediction task, with standard error over 3 seeds.

## Evidence Examined

- Paper sources:
  - `artifacts/main.tex`
  - `artifacts/A3_Experiments.tex`
  - final image files in `artifacts/figures/`
- Released repo:
  - `README.md`
  - `state_tracking/readme.md`
  - `state_tracking/src/generate_data.py`
  - `state_tracking/src/main.py`
  - `state_tracking/src/main_regression.py`
  - `state_tracking/src/utils.py`
  - `state_tracking/requirements.txt`
  - `diff_AUSSM.txt`

## Commands and Trace

### Repository state

```bash
git -C papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking rev-parse HEAD
git -C papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking status --short
```

Observed:

```text
e6575fae9d3aa8f32cf30254269054fbf58c87b1
```

No local dirty files were reported inside the cloned paper repo.

### Search for result files and plotting scripts

```bash
find papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking -maxdepth 5 \
  \( -name '*.csv' -o -name '*.json' -o -name '*.png' -o -name '*.pt' -o -name '*.bin' \
     -o -name '*.ckpt' -o -name '*.npy' -o -name '*.npz' -o -name '*.parquet' \
     -o -name '*.ipynb' -o -name '*plot*' -o -name '*figure*' -o -name '*results*' \) \
  -type f | sort
```

Observed result: the only relevant released data/result-like file in the repo was `state_tracking/src/A5.json`; there were no paper result CSVs, W&B export files, notebooks, plot scripts, checkpoints, or regenerated figure PNGs in the repository. The final paper PNGs exist only under the paper artifact directory:

```bash
file artifacts/figures/*.png
```

Observed:

```text
artifacts/figures/mse_grid.png:                 PNG image data, 5434 x 4541
artifacts/figures/mse_grid_aussm.png:           PNG image data, 7234 x 2789
artifacts/figures/seqlen_vs_depth_vertical.png: PNG image data, 2352 x 1459
```

### Paper-to-code mapping

The paper states Table 1 details in `artifacts/main.tex:572-600`, and experimental setup in `artifacts/A3_Experiments.tex:34-43`: best hyperparameter-grid results, 3 seeds, 100 epochs, 500K training sequences except `C_2`, 1000 test sequences, train length 128, test length 256.

The repo README gives only a single example command:

```bash
python src/main.py train \
  --group=${task} \
  --k=${trainLen} \
  --k_test=${evalLen} \
  --n_layers=${numLayers} \
  --epochs=100 \
  --allow_neg_eigval=True \
  --num_householder=${householder} \
  --batch_size=${batch} \
  --seed=${seed} \
  --lr=1e-3 \
  --n_heads=8 \
  --use_scheduler=True \
  --model_name=${model}
```

This is consistent with an example training entry point, but it is not a table reproduction recipe: it does not enumerate the Table 1 hyperparameter grid, all model/task/layer combinations, the selected best runs, the 3 model seeds, the data-generation seeds, or the aggregation rule used to select the displayed scalar values.

### Data generation path

The data generator is `state_tracking/src/generate_data.py`. It samples random sequences and writes `data/{group}={k}.csv`. The default data seed is random:

```text
state_tracking/src/generate_data.py:151-157
seed: int = random.randint(0, 1_000_000)
```

The paper reports fixed dataset sizes, but the README example:

```bash
python src/generate_data.py --group=S3 --k=128 --samples=500000
```

does not specify a seed. The training code then splits each loaded CSV with `train_size=0.99` by default (`state_tracking/src/main.py:421-431`, `state_tracking/src/main.py:450-479`). Therefore, if a reviewer follows the README and creates a 500K file for `k=128`, training uses about 495K examples. If the reviewer creates a 1000-sample `k=256` test file, the default split exposes only about 10 examples to `dataset_test["test"]`, not the paper's stated 1000. This is a concrete mismatch between the paper's stated sample counts and the released command path.

The only data file found in the repo was a tiny smoke-test file:

```bash
wc -l state_tracking/data/S3=4.csv
sed -n '1,5p' state_tracking/data/S3=4.csv
```

Observed:

```text
9 state_tracking/data/S3=4.csv
seed,input,target
123,5 4 2 4,5 1 4 3
123,5 0 2 2,5 5 3 5
123,0 5 3 1,0 5 2 3
123,1 0 2 0,1 1 4 4
```

No released data sufficient for Table 1 or Figures 2/3 was present.

### Figure 2 trace: maximum sequence length at 90 percent accuracy

The paper says Figure 2 uses `A_5`, depth variation, and models trained on length up to 128 (`artifacts/main.tex:645-665`). The code contains a plausible W&B metric:

```text
state_tracking/src/main.py:946-999 logs:
train/max_seq_len_at_90
val/max_seq_len_at_90
best/train_max_seq_len_at_90
best/val_max_seq_len_at_90
```

The sequence-prefix metric is defined in `state_tracking/src/utils.py:6-43`. It computes whether every token in a prefix is correct by taking `floor(cumulative_correct / cumulative_valid)` and averaging over examples. This matches a sequence-prefix exactness interpretation.

However, the release does not provide:

- the actual W&B run IDs or exports;
- a script to download W&B tables;
- a script to aggregate best depth/seed/model values into `seqlen_vs_depth_vertical.png`;
- the list of included or omitted failed deep runs;
- the command setting `strict_len=False`, which is necessary to train on lengths `2..128` via `state_tracking/src/main.py:201-220`. The README example and defaults use `strict_len=True`, which trains on exactly length 128, not "length up to 128".

Result: Figure 2 is only partially traceable to a metric name. It is not reproducible from the repo without undocumented W&B state and undocumented run selection.

### Figure 3 trace: rotation MSE vs depth

The paper describes the `A_5` rotation task in `artifacts/main.tex:681-713` and Appendix hyperparameters in `artifacts/A3_Experiments.tex:88-103`: batch size 256, learning rate `1e-3`, hidden size 128, depth 1-8, 500K training sequences of length 128, 50 epochs, 3 seeds.

The repo has a plausible training path in `state_tracking/src/main_regression.py`:

- `task="a5_regression"` is supported at `state_tracking/src/main_regression.py:603-608`.
- `A5.json` has 60 elements and a 60x60 table:

```bash
python - <<'PY'
import json
p='state_tracking/src/A5.json'
with open(p) as f:
    data=json.load(f)
print('name', data.get('name'))
print('elements', len(data.get('elements', [])))
print('table_dim', len(data.get('table', [])), len(data.get('table', [[]])[0]) if data.get('table') else 0)
for token in ['()', '(0 1 2)', '(0 1 2 3 4)']:
    print(token, data.get('elements', []).index(token) if token in data.get('elements', []) else 'missing')
PY
```

Observed:

```text
name A5
elements 60
table_dim 60 60
() 0
(0 1 2) 46
(0 1 2 3 4) 1
```

- The regression script logs per-position MSE curves to W&B as `train/sequence_errors`, `val/sequence_errors`, `train_sequence_error_table`, and `val_sequence_error_table` at `state_tracking/src/main_regression.py:1174-1197` and `state_tracking/src/main_regression.py:1272-1295`.

But the release still lacks the final reconstruction path:

- The README rotation command is incomplete: `A5json=`, `${task}`, `${batch}`, `${lr}`, and `${hsize}` are not fully defined in the snippet (`state_tracking/readme.md:83-105`).
- The code default batch size is 2048 (`state_tracking/src/main_regression.py:642-645`), while the appendix says batch size 256. A reviewer must know to override it.
- The code default `mamba_expand_rate` is 2 (`state_tracking/src/main_regression.py:623-626`), while the README snippet uses 4. The appendix does not state this setting.
- No raw per-seed MSE curves or standard errors are released.
- No script builds `mse_grid.png` or `mse_grid_aussm.png` from W&B data.
- The initial vector is hard-coded to `[1, 1, 1] / ||[1, 1, 1]||` in `GroupV0Dataset._sample_v0` (`state_tracking/src/main_regression.py:449-456`), while the paper only states "a vector on a unit sphere"; this is probably acceptable but is not documented in the paper text.

Result: Figure 3 is partially traceable to a training script and metric names, but the released repo does not let a reviewer recover the plotted curves or standard errors.

### Direct CLI dry-run

I also checked whether the released entry points are immediately discoverable in the current review environment:

```bash
cd papers/230fcebb-7586-46e3-9897-191540be9efa/repos/lie-algebra-state-tracking/state_tracking
python src/generate_data.py --help
python src/main.py --help
python src/main_regression.py --help
```

Observed for all three:

```text
ModuleNotFoundError: No module named 'fire'
```

This is not by itself a paper failure, because `requirements.txt` declares `fire>=0.5`. It does show that the repo is not directly runnable without environment setup. The larger reproducibility blocker is not this import error; it is the missing result aggregation and exact run manifest.

### Dependency trace

The repo pins or specifies some dependencies, but not a complete executable environment:

- Root README pins `flash-linear-attention` only by commit `8d25cac1e1779dad0f1e36cb033c5a9b10bf9c01`.
- `state_tracking/readme.md` states `gcc/12.2.0` and `cuda/12.4.1`.
- `state_tracking/requirements.txt` gives broad lower bounds for Python packages and Git dependencies, not exact versions.
- AUSSM instructions defer to an external repository and say the kernel length multiple is "8 (or 16? didn't check the detail)", while the code hard-codes a multiple-of-32 crop/assert path in `state_tracking/src/main.py:920-926` and `state_tracking/src/main_regression.py:1232-1239`.

This environment is sufficient as a starting point for a determined rerun, but insufficient for recovering the exact reported numbers.

## Observed Result

Outcome: **blocked / partial trace**.

I could trace the paper's empirical claims to plausible training scripts and W&B metric names, but I could not recover Table 1, Figure 2, Figure 3, or the depth-vs-error numerical claim from the released repo. The repository lacks the critical reproducibility artifacts: exact dataset seeds, exact train/test split recipe matching the paper's counts, exhaustive run grid, seed list, W&B run identifiers or exports, checkpoints, and plotting/aggregation scripts.

The final figures are present as PNGs in the paper artifact, but those are not a reproduction path. The code can plausibly generate raw metrics after substantial rerunning on A100/H100-class GPUs and external dependencies, but the released materials do not specify enough to reproduce the reported numbers or standard errors.

## Match / Partial / Mismatch / Blocked

- Table 1: **blocked**. Training code exists, but no grid runner, run manifest, exact seeds, released metrics, or table construction script. The README/default `train_size=0.99` path conflicts with the paper's stated 500K train and 1000 test sequence counts.
- Figure 2: **blocked with partial code trace**. The code logs `best/train_max_seq_len_at_90`, which likely underlies the figure, but there is no W&B export, aggregation script, depth/run inclusion manifest, or explicit `strict_len=False` recipe for "trained on length up to 128".
- Figure 3: **blocked with partial code trace**. The regression script logs per-position MSE curves, but the release does not include the per-seed curves, standard errors, plot builder, complete commands, or exact run IDs.
- Depth-vs-error empirical claim: **not independently recovered**. The code structure is consistent with the claim being testable, but the published repository does not let a reviewer recover the displayed evidence.

## Agreement With Reproducer A

Not assessable. I checked for `papers/230fcebb-7586-46e3-9897-191540be9efa/independent-reproducer-a.md` and no such report was present at the time of this pass.

## Limitations

- I did not install the full GPU stack or rerun 500K-sequence training. This was intentional: the assigned route was static code-to-paper tracing, and the core question was whether a reviewer can recover the tables/figures from the released repository.
- I did not query external future-outcome or OpenReview information.
- The absence of W&B exports in the local clone does not prove the authors did not have them; it means they were not released in the artifact available to reviewers.

## Confidence

High confidence that the released repo is insufficient to recover the reported Table/Figure values without additional undocumented author state. Medium confidence that the training scripts could be adapted to rerun approximate experiments, because the code contains plausible model, data, and metric paths, but the exact reported evidence is not independently reproducible from the release.

## Decision Impact

This is a material reproducibility weakness. The paper's empirical support for "depth systematically reduces error" rests on Table 1 and Figures 2/3, but the released repository does not provide a complete or auditable route from code and data to those artifacts. I would mark the empirical reproducibility of the central depth-vs-error claim as **weak**: the claim is plausible from the source structure and paper figures, but not independently recoverable from the released materials.
