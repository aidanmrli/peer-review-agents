# Independent Reproducer A Report

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`
Title: RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models
Role: Independent Reproducer A
Date: 2026-04-24

## Claim Attempted

I attempted the smallest meaningful reproduction of the central empirical claim:

> RoboAlign applies RL-based language-action alignment after SFT using less than 1% additional data, and this improves VLA performance over SFT/no-RL baselines on LIBERO, CALVIN, and real-world tasks.

The paper-level evidence for this claim is in:

- `artifacts/sections/abstract.tex`: reports `17.5%`, `18.9%`, and `106.6%` improvements over SFT baselines.
- `artifacts/sections/introduction.tex`: repeats the same headline and says the RL stage uses less than 1% additional data.
- `artifacts/sections/experiments.tex`: states RL uses a 12.8K BridgeV2 FAST-token subset after SFT and highlights gains in Tables `\ref{tab:libero}` and `\ref{tab:calvin}`.
- `artifacts/sections/method.tex`: defines the RL reward as `r=(r_f+r_a)/2`, where `r_f` is format correctness and `r_a` is prefix similarity to the target FAST-token sequence.
- `artifacts/sections/appendix.tex`: gives compute and data details: 8 H200 GPUs for MLLM training, about 30h SFT and 1h RL; 2 A100 GPUs for VLA action-head training; 60K LIBERO, 100K CALVIN, and 30K real-robot VLA training steps.

## Setup Used

Workspace: `/home/mila/l/lia/peer-review-agents/agent_configs/agent2`

Official artifacts inspected:

- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/paper.pdf`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/main.tex`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/*`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/*`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1`
- `papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T`

Repository commits verified:

```text
EasyR1:      dd71bbd252694f5f850213eec15795b6b88d9fea
Isaac-GR00T: 4b1dca9d88d2a0b9ea5a65aa61c82ff89f5c4f0e
```

Local runtime:

```text
Python 3.12.12
torch: missing
transformers: missing
datasets: missing
ray: missing
verl: missing
gr00t: missing
```

No usable local PyTorch/Ray/GR00T environment was present. `nvidia-smi --query-gpu=name,memory.total --format=csv,noheader` produced no GPU listing in this shell.

## Commands and Observations

### 1. Paper-claim extraction

Commands:

```bash
rg -n "RoboAlign|LIBERO|CALVIN|SFT|reinforcement|RL|<1|less than|headline|real-world|EasyR1|GR00T|Isaac" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/main.tex \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections -S

sed -n '41,95p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/method.tex
sed -n '1,90p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/appendix.tex
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Figure/prompt_template_Fast.tex
```

Observed:

- SFT data is described as 1.88M QA samples plus 400K BridgeV2 FAST-token samples, for 2.28M total fine-tuning samples in the main tables.
- RL data is a 12.8K BridgeV2 FAST-token subset, which is 0.561% of 2.28M and therefore supports the "less than 1%" data-volume arithmetic.
- The method specifies a reward but the artifact tree did not contain a RoboAlign-specific reward implementation matching the prefix-similarity FAST-token reward.
- The prompt template for RL requires `<think>...</think>` and `<answer>...</answer>` tags, but no released preprocessing script was found that constructs the 12.8K RL data in the required format.

### 2. Table arithmetic check

Commands:

```bash
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_LIBERO.tex
sed -n '1,180p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_CALVIN.tex
sed -n '1,180p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_REAL.tex
sed -n '1,180p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/SFT_vs_RL_LIBERO.tex

python - <<'PY'
libero_base=78.7
libero_ours=86.8
calvin_base=2.16
calvin_ours=2.57
real_base=32.3
real_ours=66.7
print('LIBERO relative vs RoboAlign w/o RL:', (libero_ours-libero_base)/libero_base*100)
print('CALVIN relative vs Qwen baseline Succ Len:', (calvin_ours-calvin_base)/calvin_base*100)
print('Real relative vs Qwen baseline Avg:', (real_ours-real_base)/real_base*100)
print('LIBERO averages:', sum([92.8,97.4,59.0,65.6])/4, sum([93.8,96.0,87.2,70.0])/4)
print('CALVIN avg rows:', sum([77.8,55.0,38.6,26.6,18.1])/100, sum([87.6,67.2,47.1,32.8,22.2])/100)
print('Real averages:', sum([16.7,70.8,20.8,20.8])/4, sum([87.5,58.3,70.8,50.0])/4)
print('RL data percent if SFT data = 2.28M:', 12800/2280000*100)
PY
```

Observed output:

```text
LIBERO relative vs RoboAlign w/o RL: 10.29224904701397
CALVIN relative vs Qwen baseline Succ Len: 18.981481481481467
Real relative vs Qwen baseline Avg: 106.50154798761614
LIBERO averages: 78.7 86.75
CALVIN avg rows: 2.161 2.569
Real averages: 32.275 66.65
RL data percent if SFT data = 2.28M: 0.5614035087719298
```

Interpretation:

- The data-volume claim is arithmetically supported: `12.8K / 2.28M = 0.56%`.
- The CALVIN headline is reproducible from the table if the baseline is the raw Qwen2.5VL row: `(2.57-2.16)/2.16 = 18.98%`, rounded to 18.9%.
- The real-world headline is reproducible from the table if the baseline is the raw Qwen2.5VL row: `(66.7-32.3)/32.3 = 106.5%`, rounded to 106.6%.
- The LIBERO headline is not reproducible against the most direct SFT/no-RL row in Table 1. `RoboAlign w/o RL` is 78.7 average, `RoboAlign (Ours)` is 86.8 average, giving a relative gain of only 10.3%, or an absolute gain of 8.1 points.
- The claimed 17.5% LIBERO improvement appears consistent with comparing `86.8` against raw Qwen2.5VL `73.9`: `(86.8-73.9)/73.9 = 17.46%`. That is not "over SFT baseline" as written in the abstract/introduction if "SFT baseline" means the no-RL RoboAlign SFT model.
- The reported table averages are rounded consistently.

### 3. Artifact/code discovery

Commands:

```bash
rg --files papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T |
  rg -i 'roboalign|bridge|bridgev2|calvin|libero|fast|reward|grpo|train|eval|rollout|qwen'

rg -n "RoboAlign|roboalign|BridgeV2|bridgev2|FAST|fast|LIBERO|CALVIN|reward|grpo|Qwen2.5|qwen2_5|12.8|12800|action" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T -S

sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/examples/qwen2_5_vl_7b_geo3k_grpo.sh
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/examples/config.yaml
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/verl/workers/reward/function.py
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T/examples/LIBERO/README.md
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T/examples/finetune.sh
```

Observed:

- The EasyR1 artifact appears to be an upstream/general RL framework snapshot. Its runnable example is `qwen2_5_vl_7b_geo3k_grpo.sh`, using `hiyouga/geometry3k`, not BridgeV2 FAST tokens.
- `examples/config.yaml` matches several high-level hyperparameters from the paper: rollout batch size 512, actor global batch size 128, rollout `n: 5`, learning rate `1.0e-6`, and 8 GPUs. However, it defaults to math/geometry data and `examples/reward_function/math.py`, not the paper's action-token reward.
- `verl/workers/reward/function.py` supports dynamically loading a custom reward function, so EasyR1 could in principle run RoboAlign if the authors supplied the missing reward file and dataset. They did not in the provided artifacts.
- The Isaac-GR00T artifact contains LIBERO finetuning/evaluation machinery, but it is the NVIDIA GR00T-N1.7 LIBERO recipe, not the paper's described Qwen2.5VL-7B hidden-state layer-18 backbone plus newly initialized action expert. The README's LIBERO result table reports 94-98% success rates for GR00T-N1.7-LIBERO, which is not the RoboAlign Table 1 regime.
- I found no CALVIN-specific recipe in the provided artifact tree.
- I found no script that converts a RoboAlign-trained MLLM into the frozen-backbone VLA evaluated in Tables 1-3.
- I found no released SFT checkpoint, RL checkpoint, VLA action-head checkpoint, 12.8K RL subset manifest, 400K FAST-token subset manifest, or evaluation logs.

### 4. Minimal code-path invocation

Commands:

```bash
python - <<'PY'
import importlib.util
mods=['torch','transformers','datasets','ray','verl','gr00t']
for m in mods:
    spec=importlib.util.find_spec(m)
    print(f'{m}:', 'FOUND' if spec else 'MISSING', spec.origin if spec else '')
try:
    import torch
    print('torch version:', torch.__version__)
    print('cuda available:', torch.cuda.is_available())
    print('cuda device count:', torch.cuda.device_count())
except Exception as e:
    print('torch import error:', type(e).__name__, str(e))
PY

python papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/examples/reward_function/android_gui.py
```

Observed:

```text
torch: MISSING
transformers: MISSING
datasets: MISSING
ray: MISSING
verl: MISSING
gr00t: MISSING
torch import error: ModuleNotFoundError No module named 'torch'
```

The standalone EasyR1 Android-GUI reward demo ran and printed correct toy scores, but it is unrelated to RoboAlign's FAST-token prefix-similarity reward. This only verifies that some pure-Python example code can execute locally; it does not reproduce the paper's RL training, VLA conversion, or benchmark evaluation.

## Reproduction Outcome

Outcome: **Blocked for empirical reproduction; partial arithmetic reproduction only.**

What I could reproduce:

- The less-than-1% additional RL-data claim is arithmetically consistent with `12.8K / 2.28M = 0.56%`.
- The CALVIN and real-world headline relative gains are arithmetically consistent if computed against raw Qwen2.5VL baselines, not necessarily against SFT-only/no-RL baselines.
- The paper's table averages are internally consistent after rounding.
- The EasyR1 framework can support custom reward files in principle, and its generic config overlaps with some hyperparameters claimed by RoboAlign.

What I could not reproduce:

- No RoboAlign SFT training command was provided.
- No RoboAlign RL command was provided.
- No BridgeV2 FAST-token 400K SFT subset or 12.8K RL subset was provided.
- No custom RoboAlign reward implementation was provided for the paper's format reward plus FAST-token prefix-similarity accuracy reward.
- No trained MLLM checkpoints were provided.
- No conversion code was found to transfer the trained MLLM into the VLA setup described in the paper.
- No action-head training/evaluation scripts matching the paper's LIBERO/CALVIN/real-world setup were provided.
- The local environment lacked PyTorch, Ray, EasyR1/verl, GR00T, and visible GPU support, so even upstream framework training/evaluation could not be invoked without substantial environment construction.

## Decision-Relevant Notes

1. The core empirical claim is not independently reproducible from the supplied artifacts. The repositories are mostly upstream EasyR1 and Isaac-GR00T snapshots rather than a runnable RoboAlign release.
2. The missing custom reward code is especially material. The paper's claimed novelty depends on action-token reward shaping, but the artifact only shows generic math/geometry/Android reward examples.
3. The LIBERO headline improvement is potentially misframed. The advertised `17.5%` gain matches comparison to raw Qwen2.5VL, while the text says "over SFT baselines"; comparison to `RoboAlign w/o RL` gives `10.3%` relative gain.
4. The reported gains may still be real, but this pass cannot verify them beyond table arithmetic. Under a reproducibility-first standard, the result should be treated as **weakly reproducible** until the authors provide the exact data manifests, reward implementation, training scripts, checkpoints, and evaluation logs.

## Score Impact

This reproduction attempt materially reduces confidence in the paper's central acceptance case. I would not credit the headline LIBERO/CALVIN/real-world improvements as independently reproduced. The correct characterization from this pass is: **paper-table arithmetic mostly checks out, but the claimed method-level and benchmark-level improvements are blocked by missing RoboAlign-specific artifacts and environment requirements.**
