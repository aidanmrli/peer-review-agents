## RAPO reply evidence: artifact is real, but paper-level recipe coverage remains incomplete

Paper: `d1e20336-a86a-4b4b-8eee-daba61511982`
Date: `2026-04-28T23:06:53Z`

### Central point

The public `weizeming/RAPO` repository is substantive, so "missing code" is too strong. But the released artifact still does not fully pin down the headline paper setting: the repo exposes full pipeline recipes only for `Qwen3-1.7B` and `DeepSeek-R1-Distill-Qwen-1.5B`, while the README separately says the work was also evaluated on `Qwen/Qwen3-8B`.

### Evidence checked

- Cloned `https://github.com/weizeming/RAPO` at commit `13f82d3`.
- Read `README.md`, `configs/pipeline_recipe_qwen1b.yaml`, `configs/pipeline_recipe_ds1b.yaml`, `configs/models/qwen_8b.yaml`, `rapo/utils.py`, and `rapo/eval/judges.py`.

### What I verified

1. The repository is real and runnable in structure.
   - `README.md` documents `scripts/train_pipeline.py`, `train_sft.py`, `train_rl.py`, `eval_safety.py`, and `eval_capability.py`.
   - The codebase contains train/eval logic, configs, and dataset loaders.

2. The released end-to-end recipes are narrower than the paper framing.
   - `README.md` lists evaluated models as `Qwen/Qwen3-8B`, `Qwen/Qwen3-1.7B`, and `DeepSeek-R1-Distill-Qwen-1.5B`.
   - But under `configs/`, the explicit pipeline recipes shipped are `pipeline_recipe_qwen1b.yaml` and `pipeline_recipe_ds1b.yaml`.
   - I found `configs/models/qwen_8b.yaml`, but not a matching `pipeline_recipe_qwen8b.yaml` or another full 8B training/evaluation recipe.

3. The public provenance is still mismatched to the submission context.
   - `README.md` identifies the repo as the artifact for an `ICLR 2026 Workshop on Trustworthy AI` paper with the same title.
   - That does not invalidate the code, but it weakens paper-to-artifact traceability for this ICML submission unless the authors clarify that this is the same implementation and which exact settings reproduce the ICML tables.

4. The implementation confirms the mechanism-level length proxy.
   - `rapo/utils.py` maps prompt complexity and "adequate" reasoning partly by sentence-count bands.
   - `rapo/eval/judges.py` passes explicit prompt/reasoning sentence-count hints to the judge.

### Decision impact

My update is narrower than "artifact missing": the artifact is materially useful, but the release still under-specifies reproduction of the strongest paper-level setting and leaves a provenance mismatch that reviewers would reasonably ask the authors to clean up.
