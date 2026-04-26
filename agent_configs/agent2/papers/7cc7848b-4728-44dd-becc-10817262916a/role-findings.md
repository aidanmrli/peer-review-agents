# LaRA-VLA Role Findings

## Reproducibility lead: central claim and reproduction target

Central claim audited: LaRA-VLA's latent-reasoning curriculum improves LIBERO, SimplerEnv/Bridge, and long-horizon real-robot manipulation while cutting inference latency by up to 90% relative to explicit CoT baselines. Reproduction target: verify that the released artifact exposes the named training stages, evaluation entrypoints, and the data/weights/assets needed to recover the reported benchmark tables.

## Reproducer A: artifact-first check

- Koala tarball is paper source only: `main.tex`, section files, figures, and tables; no runnable code or checkpoints.
- The project page is live and links to `https://github.com/LoveJu1y/LaRA-VLA`.
- Repo README explicitly says: `Training code is released`, `Evaluation code is released`, but `Pretrained model weights are not released yet` and `Training datasets are not released yet`.
- README reports paper numbers for LIBERO and SimplerEnv, but those results depend on absent checkpoints and non-released training data.

## Reproducer B: clean-room / specification check

- Paper claims two structured CoT datasets (`LIBERO-LaRA`, `Bridge-LaRA`) plus a real-world manipulation dataset: `sections/1-intro.tex:20`, `sections/1-intro.tex:33`, `sections/3-method.tex:18`.
- Repo scripts do expose multi-stage training and benchmark evaluation:
  - `scripts/run_bridge_multistage.sh`
  - `scripts/run_libero_multistage.sh`
  - `examples/LIBERO/README.md`
  - `examples/SimplerEnv/README.md`
- But an external clean-room run still stops at missing assets: the repo does not ship the structured CoT datasets, released checkpoints, or a documented public real-robot reproduction package.

## Implementation auditor: code/artifact/repo match

- The codebase generally matches the paper's staged-training story: `laravla/training/train.py` plus multi-stage scripts support staged latent-reasoning training.
- Important mismatch: Bridge config remains machine-specific and points to private annotation/data paths in `laravla/config/training/bridge.yaml`, including:
  - `data_root_dir: /share/project/baishuanghao/data`
  - `cot_path: /share/project/baishuanghao/data/bridge_orig_lerobot/...`
  - `bbox_path: /share/project/baishuanghao/data/bridge_orig_lerobot/...`
  - `steps_cache_path: /share/project/baishuanghao/data/bridge_orig_lerobot/...`
  - `cache_dir: /share/project/lvjing/starVLA/qwen_cache`
- The repo's own `docs/open_source_release_schedule.md` acknowledges cleanup is unfinished and still lists private-path and debug-entrypoint removal as pending release work.
- `laravla/dataloader/lerobot_datasets.py` still contains a `debugpy.listen(("0.0.0.0", 10092))` debug block under `__main__`.
- Real-robot deployment material is not reproduction-grade: `deployment/readme-deployment.md` is a short internal note with raw network/interface commands, not a runnable external protocol.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

- I did not independently verify the reported LIBERO / SimplerEnv / real-robot numbers from code execution because the released artifact lacks the required datasets and weights.
- The paper's claim that latent reasoning avoids collapse is supported only by paper figures and text, not by public checkpoints or logs that would let an external reviewer inspect learned latents.
- The inference-efficiency claim is similarly unreproducible from the artifact without checkpoints and the exact benchmark setup.

## Literature specialist: novelty/framing against permitted prior work

- Framing seems plausible: latent reasoning for VLA is a meaningful extension of latent-CoT ideas, and the code is more substantive than many paper-only releases.
- The practical acceptance case still rests on unreproduced empirical gains. For an ICML methods paper, the current artifact supports architecture inspection and partial workflow understanding, but not independent confirmation of the headline results.
