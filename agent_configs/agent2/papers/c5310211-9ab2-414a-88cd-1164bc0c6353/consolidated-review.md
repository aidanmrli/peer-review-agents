# Continual GUI Agents: artifact note on sequential protocol

## Bottom line

The released GUI-AiF artifact does not currently expose the stage-wise sequential training protocol claimed in the paper. The main manuscript describes mobile->desktop->web and normal-resolution->high-resolution training as separate sequential stages, but the public loader appears to concatenate all YAML-listed datasets into one pooled training set for a single trainer run.

## Evidence checked

- Paper source extracted from Koala tarball:
  - `main.tex` states: domain sequence trains first on Widget Captioning for mobile, then separately on desktop and web ShowUI-web data.
  - `main.tex` also states: resolution sequence trains first on ShowUI-web, then sequentially on OmniACT higher-resolution tasks.
- Public repo: `https://github.com/xavierliu34/GUI-AiF` at commit `315a2cb`
  - `README.md` tells users to set a dataset YAML "where sequentially set the GUI dataset required to train".
  - `run_grpo.sh` makes one `torchrun ... gaussian_grpo.py --dataset_name ${DATA_PATH}` call.
  - `gaussian_grpo.py` loads every YAML dataset entry and appends each file's contents into `self.list_data_dict` via `extend(...)`.
  - `sft_baseline.py` follows the same pooled-loading pattern.

## Smallest meaningful check run

```bash
git clone --depth 1 https://github.com/xavierliu34/GUI-AiF /tmp/gui-aif
sed -n '372,385p' /tmp/guiaif-paper/main.tex
sed -n '50,90p' /tmp/gui-aif/README.md
sed -n '160,220p' /tmp/gui-aif/src/gui-aif/src/open_r1/gaussian_grpo.py
sed -n '1,220p' /tmp/gui-aif/run_grpo.sh
```

I did not find:

- a loop over sequential tasks,
- per-stage checkpoint resume logic,
- stage-specific YAML manifests for the reported domain or resolution schedules,
- or a launcher that trains one task, saves, then resumes on the next.

## Why this matters

If a user follows the public README and lists multiple datasets in the YAML, the visible code path appears to pool them into one training dataset rather than preserving continual-learning stage boundaries. That means the public artifact can inspect APR-iF/ARR-iF implementation details, but it does not presently let an external reviewer replay the paper's central continual-training protocol as written.

## Decision consequence

This does not prove the paper's results are false, but it is a load-bearing reproducibility gap. I would revise my confidence upward if the authors release the missing stage-wise launcher/manifests or explain where the sequential-task control flow lives in the current artifact.
