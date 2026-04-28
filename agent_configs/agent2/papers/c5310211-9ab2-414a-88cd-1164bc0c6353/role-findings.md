## Central claim and reproduction target

The paper claims stage-wise continual GUI training: for domain flux, train on mobile, then separately on desktop and web; for resolution flux, train on normal-resolution ShowUI-web, then sequentially on OmniACT higher-resolution tasks (`main.tex`, Continual GUI Agents Implementation). My reproduction target was narrower than rerunning training: verify whether the released artifact exposes that sequential protocol at all.

## Paper and artifact evidence checked

- Paper source: `/tmp/guiaif-paper/main.tex`
  - Continual protocol text: lines around the "Continual GUI Agents Implementation" subsection say the tasks are trained "separately" and "sequentially".
  - Training details say all experiments use one epoch on 4 A100-80G GPUs with `alpha=15`, `gamma=0.5`.
- Code artifact: `https://github.com/xavierliu34/GUI-AiF`, cloned at commit `315a2cb`.
  - `README.md` says `DATA_PATH` is a dataset YAML "where sequentially set the GUI dataset required to train".
  - `run_grpo.sh` passes one `--dataset_name ${DATA_PATH}` into a single `torchrun ... gaussian_grpo.py` call.
  - `src/gui-aif/src/open_r1/gaussian_grpo.py` defines `LazySupervisedDataset`, iterates over every YAML `datasets` entry, loads each file, and `self.list_data_dict.extend(cur_data_dict)`.
  - `src/gui-aif/src/open_r1/sft_baseline.py` uses the same pattern for SFT.

## Reproducibility result from the smallest meaningful check you actually ran

I ran a static protocol check rather than a training run:

```bash
git clone --depth 1 https://github.com/xavierliu34/GUI-AiF /tmp/gui-aif
sed -n '372,385p' /tmp/guiaif-paper/main.tex
sed -n '50,90p' /tmp/gui-aif/README.md
sed -n '160,220p' /tmp/gui-aif/src/gui-aif/src/open_r1/gaussian_grpo.py
sed -n '1,220p' /tmp/gui-aif/run_grpo.sh
```

Result: the public training path does not encode stage boundaries. A multi-entry YAML is flattened into one pooled `list_data_dict`, then consumed by one trainer invocation for one epoch. I did not find a loop over tasks, a per-stage checkpoint handoff, or stage-specific config manifests that would reproduce the paper's sequential mobile->desktop->web or normal->high-resolution schedule.

## Implementation or correctness risks

- If users follow the public README literally and add multiple datasets to the YAML, the released code appears to train on the union of those datasets, not on sequential stages. That is a different experimental protocol from continual learning.
- Because the artifact does not preserve task boundaries, it cannot test whether reported gains come from continual adaptation versus simple pooled multi-domain fine-tuning.
- This issue is orthogonal to earlier setup-path complaints: even after fixing absolute paths and missing helpers, the current public control flow still leaves the stage-wise protocol underspecified.

## Novelty/framing context from permitted prior work

This check does not challenge the task framing itself. It only challenges whether the released artifact instantiates the framing described in the manuscript.

## Decision impact

This materially lowers reproducibility confidence. The public code is useful for inspecting reward formulas, but it is not sufficient to reproduce the key "continual" part of the reported training protocol without additional author clarification or missing scripts/configs.
