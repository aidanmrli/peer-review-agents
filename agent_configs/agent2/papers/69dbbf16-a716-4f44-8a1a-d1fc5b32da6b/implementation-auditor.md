# Implementation Auditor Report

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`
Title: `RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models`
Role: Implementation Auditor
Date: 2026-04-24

## Bottom Line

The released artifacts do not contain a RoboAlign implementation sufficient to reproduce the paper's central empirical claims. They contain the LaTeX source plus two unmodified upstream-style repositories, `hiyouga/EasyR1` and `NVIDIA/Isaac-GR00T`, but I found no paper-specific SFT/RL data generation pipeline, FAST-token vocabulary extension code, RoboAlign reward function, EasyR1 training config for BridgeV2 action-token RL, Qwen2.5VL/Qwen3-VL RoboAlign checkpoints, GR00T conversion patches for Qwen2.5VL hidden layer 18, CALVIN or real-robot evaluation scripts, or table reproduction scripts. This is a high-severity reproducibility failure because the headline claims depend directly on these missing components.

## Artifact Inventory

- Paper PDF: `artifacts/paper.pdf`
- LaTeX source bundle: `artifacts/main.tex`, `artifacts/sections/*.tex`, `artifacts/resources/*.tex`, `artifacts/Figure/*.tex`, `artifacts/PDF/*.pdf`, `artifacts/source.tar.gz`
- EasyR1 clone: `artifacts/EasyR1`, commit `dd71bbd`, remote `https://github.com/hiyouga/EasyR1.git`
- Isaac-GR00T clone: `artifacts/Isaac-GR00T`, commit `4b1dca9`, remote `https://github.com/NVIDIA/Isaac-GR00T.git`
- No model checkpoint files were present under the artifact tree. Search for `*.safetensors`, `*.bin`, `*.pt`, `*.pth`, checkpoint-like names, parquet/csv/jsonl data, and paper-specific YAMLs only found generic repo files and no RoboAlign checkpoints or datasets.

## Paper Claim Being Tested

The implementation audit focused on whether the released code supports these paper claims:

- Stage 1 SFT trains Qwen2.5VL-7B-Instruct to generate FAST action tokens through zero-shot reasoning, using 1.88M MLLM/embodied QA samples plus 400K BridgeV2 FAST-token samples (`sections/method.tex:14-39`, `sections/experiments.tex:22-25`, `sections/appendix.tex:19`).
- Stage 2 RL uses EasyR1/GRPO on a 12.8K BridgeV2 FAST-token subset with outputs in `<think>...</think><answer>...</answer>` format and reward `r = (r_f + r_a)/2`, where `r_a` is prefix similarity between generated and target FAST-token sequences (`sections/method.tex:41-57`, `sections/experiments.tex:52-56`).
- VLA conversion attaches a newly initialized diffusion action head to a frozen MLLM backbone using GR00T-N1.5, taking hidden states from the 18th layer of Qwen2.5VL-7B-Instruct, with benchmark-specific training steps of 60K for LIBERO, 100K for CALVIN, and 30K for real robot (`sections/experiments.tex:19-20`, `sections/appendix.tex:12-15`).
- Reported results include large gains on LIBERO, CALVIN, and real robot tables, for example LIBERO average 78.7 to 86.8 after RL, CALVIN average sequence length 1.89 to 2.57, and real robot 55.2 to 66.7 (`resources/VLA_LIBERO.tex`, `resources/VLA_CALVIN.tex`, `resources/VLA_REAL.tex`).

## Code Paths Inspected

Commands and searches used:

- `rg -n "RoboAlign|roboalign|BridgeV2|FAST|action_start|think|LIBERO|CALVIN|Qwen2.5|Qwen3|GRPO|reward|accuracy|diffusion|hidden|18" artifacts/EasyR1 artifacts/Isaac-GR00T`
- `find artifacts -maxdepth 3 -type f` and targeted searches for checkpoint/data/config extensions.
- `git -C artifacts/EasyR1 log --oneline -3`, `git -C artifacts/EasyR1 remote -v`
- `git -C artifacts/Isaac-GR00T log --oneline -3`, `git -C artifacts/Isaac-GR00T remote -v`

Specific files inspected:

- Paper/source: `sections/method.tex`, `sections/experiments.tex`, `sections/appendix.tex`, `resources/VLA_LIBERO.tex`, `resources/VLA_CALVIN.tex`, `resources/VLA_REAL.tex`, `resources/SFT_vs_RL_LIBERO.tex`, `resources/VLA_LIBERO_change_alignment.tex`
- EasyR1: `examples/config.yaml`, `examples/reward_function/r1v.py`, `examples/reward_function/math.py`, `examples/format_prompt/r1v.jinja`, `examples/qwen2_5_vl_7b_geo3k_grpo.sh`, `verl/workers/reward/function.py`, `requirements.txt`
- GR00T: `examples/finetune.sh`, `examples/LIBERO/README.md`, `gr00t/configs/model/gr00t_n1d7.py`, `gr00t/model/modules/qwen3_backbone.py`, `gr00t/model/gr00t_n1d7/gr00t_n1d7.py`, `gr00t/eval/rollout_policy.py`, `pyproject.toml`

## Paper-to-Code Matches

- EasyR1 does support generic GRPO training. The generic config has `algorithm.adv_estimator: grpo`, rollout `n: 5`, actor global batch size 128, rollout batch size 512, and learning rate `1.0e-6` (`artifacts/EasyR1/examples/config.yaml:12`, `:24`, `:36`, `:54`, `:67`). These values partially match the paper's RL hyperparameter description in `sections/experiments.tex:56`.
- EasyR1 supports custom reward-function loading (`artifacts/EasyR1/verl/workers/reward/function.py`), so the framework could in principle host a RoboAlign reward.
- The generic R1-V reward file uses a `<think>...</think><answer>...</answer>` format reward (`artifacts/EasyR1/examples/reward_function/r1v.py:26-29`), which partially matches the paper's required reasoning/answer format.
- Isaac-GR00T provides a diffusion/flow-matching action-head training and rollout framework. Its config has a frozen backbone by default (`tune_llm=False`, `tune_visual=False`) and trainable action-head components (`tune_diffusion_model=True`) (`artifacts/Isaac-GR00T/gr00t/configs/model/gr00t_n1d7.py:43-47`, `:111-114`).
- Isaac-GR00T includes a LIBERO example workflow and success-rate rollout script (`artifacts/Isaac-GR00T/examples/LIBERO/README.md`, `artifacts/Isaac-GR00T/gr00t/eval/rollout_policy.py:522`).

These are generic framework matches only. They do not establish that RoboAlign's actual experiments were released.

## Paper-to-Code Discrepancies

### 1. No RoboAlign-specific EasyR1 reward

The paper's central RL objective requires prefix similarity over FAST action-token sequences normalized by target length (`sections/method.tex:49-55`). I found no implementation of this reward in EasyR1. The available rewards are generic math/R1-V/android examples. The R1-V reward grades a final answer with `mathruler.grade_answer` and returns binary accuracy (`artifacts/EasyR1/examples/reward_function/r1v.py:32-42`), not FAST-token prefix accuracy. The default config points to the math reward (`artifacts/EasyR1/examples/config.yaml:89-90`).

Severity: high. Without the exact reward, the reported RL training process and the claim that RL improves action-token alignment cannot be independently reproduced.

### 2. No SFT data construction or FAST-token vocabulary extension code

The paper says the authors add `<ACTION_START>`, `<ACTION_END>`, and 2K FAST tokens to the MLLM vocabulary, then construct BridgeV2 QA data (`sections/method.tex:34-39`). The artifact contains no script for tokenization, vocabulary resizing, BridgeV2-to-FAST conversion, prompt generation, action chunking into FAST tokens, train/validation splits, or filtering. Searches for `ACTION_START`, `<|action_`, `FAST`, and BridgeV2-specific generation logic in both repos found only paper LaTeX/examples or generic unrelated references, not a usable pipeline.

Severity: high. The SFT stage is a prerequisite for the RL stage; its absence blocks reproduction of every RoboAlign model variant.

### 3. No RoboAlign EasyR1 training configs

The paper specifies Qwen2.5VL-7B-Instruct, all-parameter RL, rollout batch 512, update batch 128, 5 samples per prompt, constant LR `1e-6`, one epoch, and 12.8K BridgeV2 samples (`sections/experiments.tex:52-56`). The released EasyR1 config is a generic `hiyouga/math12k` config with `Qwen/Qwen2.5-7B-Instruct`, `math.jinja`, and `math.py` reward (`artifacts/EasyR1/examples/config.yaml:1-15`, `:43-54`, `:89-96`). Generic Geo3K scripts reference Qwen2.5-VL but not RoboAlign data, reward, prompts, or checkpoints.

Severity: high. A reviewer cannot run the claimed RL experiment from the released files.

### 4. GR00T code does not match the paper's Qwen2.5VL-7B / layer-18 VLA conversion

The paper says the VLA action expert consumes hidden states from the 18th layer of Qwen2.5VL-7B-Instruct (`sections/appendix.tex:13-14`). The released GR00T config is for `nvidia/Cosmos-Reason2-2B`, a Qwen3-VL architecture, with `select_layer=12` and `backbone_embedding_dim=2048` (`artifacts/Isaac-GR00T/gr00t/configs/model/gr00t_n1d7.py:31-47`). The Qwen3 backbone truncates layers until `select_layer` and then returns the final hidden states (`artifacts/Isaac-GR00T/gr00t/model/modules/qwen3_backbone.py:80-90`, `:143-144`). I found no Qwen2.5VL backbone adapter, no config setting `select_layer=18`, and no bridge from a RoboAlign SFT/RL Qwen2.5VL checkpoint into the GR00T model.

Severity: high. The released GR00T code supports a related generic architecture but does not instantiate the paper's stated VLA conversion.

### 5. LIBERO scripts are generic and use different training/evaluation settings

The paper reports LIBERO VLA training for 60K steps and evaluation over 50 trials per task / 500 trials per category (`sections/experiments.tex:36-38`, `sections/appendix.tex:15`). The included GR00T LIBERO README trains NVIDIA's `GR00T-N1.7-3B` for 20K steps with global batch size 640 and evaluates example commands with `--n-episodes 10` on one task (`artifacts/Isaac-GR00T/examples/LIBERO/README.md:37-45`, `:60-67`, `:80-87`, `:100-107`, `:136-145`). It also advertises NVIDIA's own LIBERO success rates around 94-98 percent (`README.md:17-22`), which are not the paper's table values. There is no script to aggregate 40 LIBERO tasks into the table values in `resources/VLA_LIBERO.tex`.

Severity: high for table reproduction. The generic LIBERO example is not a reproduction script for RoboAlign's reported table.

### 6. No CALVIN implementation or evaluation path

The paper reports CALVIN ABC-to-D training/evaluation and table results (`sections/experiments.tex:42-45`, `resources/VLA_CALVIN.tex`). I found no CALVIN-specific code, environment wrapper, dataset converter, finetune config, rollout script, or metric aggregation in the released GR00T or EasyR1 artifacts.

Severity: high. The CALVIN claim is unsupported by released artifacts.

### 7. No real-robot dataset, training, or evaluation scripts for the reported tasks

The paper reports four Franka Research 3 pick-and-place tasks, 60 demonstrations per task, 24 trials per object, 30K VLA training steps, and table results (`sections/experiments.tex:79`, `resources/VLA_REAL.tex`). The released GR00T repo contains generic real-world deployment/DROID/SO100 materials, not the paper's FR3 task setup, data schema, robot control client, evaluation protocol, or success labels.

Severity: high. The real-world improvement claim is not independently auditable from the artifact.

### 8. No checkpoints or model cards for any reported model variant

The paper compares base Qwen2.5VL, language-only SFT, action-only SFT, RoboAlign SFT, RoboAlign SFT+RL, Qwen3-VL variants, ECoT SFT, language-RL, and visual-RL variants. No corresponding checkpoints or Hugging Face model IDs are present. The GR00T README references NVIDIA's public `GR00T-N1.7-LIBERO`, not RoboAlign-derived models (`artifacts/Isaac-GR00T/examples/LIBERO/README.md:119-134`).

Severity: high. Without checkpoints, even inference-only verification of table values is blocked.

### 9. Environment is only partially pinned

GR00T has a tightly pinned `pyproject.toml` and `uv.lock`, which is useful (`artifacts/Isaac-GR00T/pyproject.toml:10-53`). EasyR1 dependencies are mostly unpinned ranges (`artifacts/EasyR1/requirements.txt:1-20`) and there is no RoboAlign-specific environment file tying EasyR1, Qwen2.5VL SFT, FAST tokenization, BridgeV2 conversion, and GR00T conversion together.

Severity: medium. This would be manageable if the scripts and data were present, but it compounds the missing pipeline.

## Reproducibility Blockers

- Missing custom RoboAlign VQA data generation scripts, Gemini prompting, metadata schema, and filtering rules.
- Missing reasoning distillation model training code and generated 76K reasoning dataset.
- Missing BridgeV2/DROID Robot QA construction and FAST-token conversion scripts.
- Missing tokenizer/vocabulary extension code for action markers and 2K FAST tokens.
- Missing SFT training command/config for Qwen2.5VL-7B-Instruct and Qwen3-VL-8B-Instruct.
- Missing RL dataset split/list for the 12.8K or 5K BridgeV2 subsets.
- Missing RoboAlign GRPO reward function implementing format plus prefix action-token accuracy.
- Missing RoboAlign EasyR1 config or launch script.
- Missing model checkpoints for SFT, SFT+RL, baselines, and ablations.
- Missing Qwen2.5VL-to-GR00T conversion code using hidden layer 18.
- Missing LIBERO/CALVIN/real-robot configs with the paper's training steps, seeds, batch sizes, evaluation task lists, and aggregation scripts.
- Missing table reproduction scripts for LIBERO, CALVIN, real robot, KNN analysis, MLLM benchmarks, and ablations.

## Commands and Environment Details

Audit was static because runnable reproduction artifacts were absent. The key commands were:

```bash
sed -n '1,220p' skills/implementation-auditor.md
find papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts -maxdepth 3 -type f | sort
rg -n "RoboAlign|roboalign|BridgeV2|FAST|action_start|think|LIBERO|CALVIN|Qwen2.5|Qwen3|GRPO|reward|accuracy|diffusion|hidden|18" papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 log --oneline -3
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T log --oneline -3
find papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts -maxdepth 3 \( -iname '*ckpt*' -o -iname '*checkpoint*' -o -iname '*.safetensors' -o -iname '*.bin' -o -iname '*.pt' -o -iname '*.pth' -o -iname '*.yaml' -o -iname '*.jsonl' -o -iname '*.parquet' -o -iname '*.csv' \) -type f | sort
```

No training or evaluation command was run because no paper-specific script, config, data, or checkpoint exists to execute without reconstructing the authors' private pipeline.

## Final Synthesis and Score Impact

From an implementation-audit perspective, the artifact release supports only the statement that the authors may have built on EasyR1 and GR00T-like frameworks. It does not support independent verification of RoboAlign itself. The most decision-relevant missing item is the exact action-token prefix reward and BridgeV2 FAST-token RL pipeline; the second is the VLA conversion code matching Qwen2.5VL layer-18 hidden states and the reported LIBERO/CALVIN/real-robot evaluation settings.

This should materially reduce confidence in the paper. The claimed improvements on LIBERO, CALVIN, and real robot are not reproducible from released code/configs/checkpoints, and multiple central claims are currently unverifiable rather than merely expensive to rerun. My implementation-audit severity is **high** and should push the overall review toward rejection unless other roles find unusually strong independent evidence.
