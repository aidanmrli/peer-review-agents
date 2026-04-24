# Independent Reproducer B Report

Paper: 69dbbf16-a716-4f44-8a1a-d1fc5b32da6b
Title: RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models
Role: Independent Reproducer B
Date: 2026-04-24

## Claim Attempted

I attempted to independently validate the paper's central empirical claim through a code-level trace and table/statistical consistency route rather than rerunning the full robotics pipeline:

> RoboAlign's SFT+RL alignment on FAST action tokens improves VLA performance over the SFT/base baselines by 17.5% on LIBERO, 18.9% on CALVIN, and 106.6% on real-robot tasks, using less than 1% additional RL data, with reproducible machinery based on EasyR1 for GRPO and Isaac-GR00T for VLA training/evaluation.

Relevant paper locations:

- `artifacts/sections/introduction.tex`, lines found by `rg`: claimed 17.5%, 18.9%, and 106.6% improvements.
- `artifacts/sections/method.tex`: Stage 2 reward definition, format reward plus FAST-token prefix-similarity accuracy reward.
- `artifacts/sections/experiments.tex`: datasets, baselines, EasyR1 RL settings, LIBERO/CALVIN/real robot descriptions.
- `artifacts/sections/appendix.tex`: VLA implementation details, GR00T-N1.5 claim, 18th-layer Qwen2.5VL hidden states, training steps, dataset composition.
- `artifacts/resources/VLA_LIBERO.tex`, `VLA_CALVIN.tex`, `VLA_REAL.tex`, `VLA_LIBERO_change_alignment.tex`, `SFT_vs_RL_LIBERO.tex`, and `knn.tex`: reported result tables.

## Independent Route Used

I used an artifact sufficiency and consistency route:

1. Identify the exact reported headline numbers from LaTeX tables.
2. Recompute relative improvements from table entries.
3. Trace whether the cloned EasyR1 artifact contains a RoboAlign-specific reward, prompt, data recipe, or run config implementing the paper's FAST-token prefix-similarity reward.
4. Trace whether the cloned Isaac-GR00T artifact contains the claimed VLA setup: GR00T-N1.5, Qwen2.5VL-7B hidden states from layer 18, LIBERO/CALVIN/real-robot training/evaluation scripts, RoboAlign checkpoints, metrics, prediction logs, or configs.
5. Check whether any raw metrics/checkpoints/logs exist locally that would permit recomputation of a headline success rate without rerunning the full experiment.

I did not read Independent Reproducer A's report before writing this report.

## Commands and Trace Steps

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent2
```

Role instructions read:

```bash
sed -n '1,220p' skills/independent-reproducer-b.md
```

Artifact inventory:

```bash
find papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b -maxdepth 3 -type f | sort
find papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts -maxdepth 2 -type d | sort
```

Claim and table extraction:

```bash
rg -n "RoboAlign|reward|LIBERO|CALVIN|GR00T|EasyR1|alignment|test-time|reason|Table|success|checkpoint|code|repository|GitHub|github" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/main.tex \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources

sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_LIBERO.tex
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_CALVIN.tex
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_REAL.tex
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_LIBERO_change_alignment.tex
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/SFT_vs_RL_LIBERO.tex
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/knn.tex
```

Headline arithmetic:

```bash
awk 'BEGIN{
  printf "LIBERO relative gain vs w/o RL: %.6f%%\n", (86.8-78.7)/78.7*100;
  printf "CALVIN relative gain vs w/o RL: %.6f%%\n", (2.57-1.89)/1.89*100;
  printf "CALVIN relative gain vs base Qwen: %.6f%%\n", (2.57-2.16)/2.16*100;
  printf "Real relative gain vs w/o RL: %.6f%%\n", (66.7-55.2)/55.2*100;
  printf "Real relative gain vs base Qwen: %.6f%%\n", (66.7-32.3)/32.3*100;
  printf "RL data fraction vs 2.28M: %.6f%%\n", 12800/2280000*100;
  printf "RL data fraction vs 1.88M: %.6f%%\n", 12800/1880000*100;
}'
```

Observed output:

```text
LIBERO relative gain vs w/o RL: 10.292249%
CALVIN relative gain vs w/o RL: 35.978836%
CALVIN relative gain vs base Qwen: 18.981481%
Real relative gain vs w/o RL: 20.833333%
Real relative gain vs base Qwen: 106.501548%
RL data fraction vs 2.28M: 0.561404%
RL data fraction vs 1.88M: 0.680851%
```

Repository identity:

```bash
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 remote -v
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 rev-parse HEAD
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T remote -v
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T rev-parse HEAD
```

Observed commits:

```text
EasyR1 origin: https://github.com/hiyouga/EasyR1.git
EasyR1 HEAD: dd71bbd252694f5f850213eec15795b6b88d9fea
Isaac-GR00T origin: https://github.com/NVIDIA/Isaac-GR00T.git
Isaac-GR00T HEAD: 4b1dca9d88d2a0b9ea5a65aa61c82ff89f5c4f0e
```

Search for RoboAlign-specific implementation, configs, logs, checkpoints, and metrics:

```bash
rg -n "RoboAlign|roboalign|LIBERO|libero|CALVIN|calvin|GR00T|gr00t|reward|alignment|vla|Bridge|bridge|task suite|success rate|checkpoint|ckpt|wandb|metrics|eval" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T

find papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts -type f \
  \( -name '*.safetensors' -o -name '*.bin' -o -name '*.pt' -o -name '*.pth' -o -name '*.ckpt' -o -name '*.parquet' -o -name '*.jsonl' -o -name '*.csv' -o -name '*.npy' -o -name '*.npz' -o -name '*metrics*' -o -name '*result*' -o -name '*log*' \) | sort

du -ah papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts | sort -h | tail -60
```

Reward-function trace:

```bash
sed -n '1,130p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/examples/reward_function/r1v.py
sed -n '1,150p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/verl/workers/reward/function.py
sed -n '1,150p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1/examples/config.yaml
```

GR00T architecture/config trace:

```bash
rg -n "N1\.5|n1\.5|Qwen2|18-th|18th|layer.*18|hidden_states|hidden state|GR00T-N1\.5|N1\.7|Qwen3|Cosmos" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/appendix.tex \
  --glob '!ATTRIBUTIONS.md' --glob '!uv.lock'

sed -n '1,190p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T/gr00t/configs/model/gr00t_n1d7.py
sed -n '1,190p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T/gr00t/model/modules/qwen3_backbone.py
sed -n '1,220p' papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T/examples/LIBERO/README.md
```

## Findings

### 1. Table arithmetic is internally reproducible, but it clarifies the denominator ambiguity

The reported LIBERO, CALVIN, and real-robot headline percentages can be partially reconstructed from the tables, but not consistently as "over SFT baselines."

- LIBERO: Table `VLA_LIBERO.tex` reports base Qwen2.5VL average 73.9 and RoboAlign 86.8. `(86.8 - 73.9) / 73.9 = 17.46%`, matching the abstract/introduction's 17.5%. However, against the paper's own RoboAlign SFT/w/o-RL row, the gain is only `(86.8 - 78.7) / 78.7 = 10.29%`.
- CALVIN: Table `VLA_CALVIN.tex` reports base Qwen average sequence length 2.16 and RoboAlign 2.57. `(2.57 - 2.16) / 2.16 = 18.98%`, matching 18.9%. Against RoboAlign w/o RL, the gain is 35.98%, not 18.9%.
- Real robot: Table `VLA_REAL.tex` reports base Qwen average 32.3 and RoboAlign 66.7. `(66.7 - 32.3) / 32.3 = 106.50%`, matching 106.6%. Against RoboAlign w/o RL, the gain is 20.83%.
- The "less than 1% additional data" claim is arithmetically supported: `12.8K / 2.28M = 0.561%` and `12.8K / 1.88M = 0.681%`.

This is a table-consistency match, not an empirical reproduction. The paper text should be more precise about whether the denominator is the original Qwen baseline or the SFT/no-RL baseline.

### 2. The EasyR1 clone does not expose a RoboAlign FAST-token reward implementation

The paper's method defines the RL reward as:

- format reward for `<think>...</think>` and answer tags;
- FAST-token prefix similarity against target action-token sequence;
- final reward `(r_f + r_a) / 2`.

In the cloned EasyR1 artifact, I found only generic upstream reward examples. The closest file, `artifacts/EasyR1/examples/reward_function/r1v.py`, implements exact-answer grading via `mathruler.grader.grade_answer`, not prefix similarity over FAST action tokens:

```python
def format_reward(response: str) -> float:
    pattern = re.compile(r"<think>.*?</think>\s*<answer>.*?</answer>", re.DOTALL)
    format_match = re.fullmatch(pattern, response)
    return 1.0 if format_match else 0.0

def accuracy_reward(response: str, ground_truth: str) -> float:
    ...
    if grade_answer(given_answer, ground_truth.strip()):
        return 1.0
```

The EasyR1 reward manager supports loading custom reward files (`verl/workers/reward/function.py`), but no RoboAlign-specific custom reward file, FAST-token parser, prefix-similarity scorer, BridgeV2 action-token dataset recipe, or training invocation was present in the artifact. The provided `examples/config.yaml` is generic math/RL configuration (`hiyouga/math12k`, `Qwen/Qwen2.5-7B-Instruct`, `reward_function: ./examples/reward_function/math.py:compute_score`), not the paper's Qwen2.5VL-7B FAST-token setup.

Consequence: I could verify that EasyR1 could host a custom reward, but I could not verify that the paper's actual reward was implemented or used.

### 3. The Isaac-GR00T clone is not the claimed GR00T-N1.5/Qwen2.5VL VLA implementation

The paper appendix says:

- implementation refers to GR00T-N1.5;
- the action expert consumes hidden states from the 18th layer of Qwen2.5VL-7B-Instruct;
- action experts are trained for LIBERO, CALVIN, and real robot with benchmark-specific step counts.

The cloned `artifacts/Isaac-GR00T` repository is upstream Isaac-GR00T at commit `4b1dca9d88d2a0b9ea5a65aa61c82ff89f5c4f0e`, and its README/configs are for GR00T N1.7. `gr00t/configs/model/gr00t_n1d7.py` states that N1.7 uses `nvidia/Cosmos-Reason2-2B` with Qwen3-VL architecture, not Qwen2.5VL-7B. The default config has `select_layer: int = 12`, not the paper's 18th-layer claim. `gr00t/model/modules/qwen3_backbone.py` loads `Qwen3VLForConditionalGeneration` and returns `outputs.hidden_states[-1]` after pruning layers to `select_layer`; it is not an exposed Qwen2.5VL-7B layer-18 feature extractor.

The LIBERO README in this cloned repo reports NVIDIA GR00T-N1.7 benchmark commands and results, e.g. `nvidia/GR00T-N1.7-3B`, `nvidia/GR00T-N1.7-LIBERO`, 20K steps, and batch size 640. These are not the paper's reported VLA setup of 60K LIBERO steps, 100K CALVIN steps, batch size 32, and Qwen2.5VL hidden states.

Consequence: the provided GR00T clone is useful as an upstream reference but not sufficient to reproduce the RoboAlign VLA conversion or evaluation.

### 4. No raw metrics, checkpoints, prediction logs, or RoboAlign datasets were available

The local artifact tree is only 28 MB total. The only data-like files I found were tiny GR00T demo parquets/jsonl files and demo masks under `artifacts/Isaac-GR00T/demo_data`. There were no RoboAlign model checkpoints (`*.safetensors`, `*.bin`, `*.pt`, `*.pth`, `*.ckpt`), no BridgeV2 12.8K RL subset, no SFT mixture manifest, no FAST-token vocabulary/data conversion files, no CALVIN scripts/results, no real-robot logs, no LIBERO per-task result logs, and no KNN representation features or labels.

The `find` command found demo files such as:

- `artifacts/Isaac-GR00T/demo_data/libero_demo/data/chunk-000/episode_000000.parquet` through `episode_000004.parquet`
- `artifacts/Isaac-GR00T/demo_data/libero_demo/meta/*.json*`
- generic EasyR1 and GR00T code files

It did not find any paper-specific metric or checkpoint artifact that could independently recompute Table 1, Table 2, Table 3, the alignment ablations, or the KNN result.

### 5. Evaluation scripts are incomplete for the paper's full benchmark claim

The GR00T clone includes generic simulation rollout machinery for LIBERO and SimplerEnv. I found no CALVIN evaluation path in the cloned repository, despite CALVIN being one of the three headline claims. The paper gives aggregate CALVIN numbers over 1,000 chains, but the artifacts do not expose the exact environment setup, task seeds/chains, policy checkpoint, action-head config, or evaluation logs required to reproduce the reported `2.57` average success length.

For LIBERO, upstream GR00T can evaluate a checkpoint, but the provided scripts point to NVIDIA N1.7 checkpoints or user-provided local checkpoints. No RoboAlign checkpoint path is supplied.

For real robot, the paper reports 96 trials per task/condition, but no trial logs, videos, success criteria, randomization protocol, model checkpoints, or hardware execution scripts specific to the four reported tasks are provided.

## Observed Result

Outcome: **blocked empirical reproduction with partial table-arithmetic match**.

What I could reproduce:

- The headline percentages are arithmetically recoverable from the LaTeX tables if the denominator is the base Qwen2.5VL row for LIBERO/CALVIN/real robot.
- The "less than 1%" RL-data claim is arithmetically recoverable from the stated `12.8K` and `2.28M`/`1.88M` sample counts.
- EasyR1 contains generic GRPO infrastructure and supports custom reward functions.
- Isaac-GR00T contains generic VLA training/evaluation infrastructure.

What I could not reproduce:

- The actual FAST-token prefix-similarity reward used by RoboAlign.
- The SFT data mixture, 400K FAST-token dataset, or 12.8K RL subset.
- Any RoboAlign SFT/RL training run.
- Any RoboAlign VLA action-head training run.
- Any LIBERO/CALVIN/real-robot success rate from raw predictions or logs.
- The KNN representation-analysis result.
- The paper's stated Qwen2.5VL-7B layer-18 GR00T-N1.5 VLA implementation, because the provided GR00T repository is N1.7/Cosmos-Qwen3 oriented.

## Match / Partial Match / Mismatch / Blocked

Classification: **Blocked, with partial table/statistical consistency match**.

The core claim is not independently reproducible from the supplied artifacts. The artifacts are upstream framework snapshots plus paper source, not a reproducibility package for RoboAlign. The largest blockers are missing paper-specific code/configs/reward functions, missing datasets/subset manifests, missing checkpoints, missing raw evaluation logs, and an apparent mismatch between the paper's described GR00T-N1.5/Qwen2.5VL layer-18 implementation and the supplied GR00T-N1.7/Qwen3-Cosmos code.

## Reproducibility Impact

Under agent2's reproducibility standard, this is **weak reproducibility**. The paper's central empirical claim depends on large training and robotics evaluation runs, but the artifact does not let an independent reviewer reproduce or audit those runs beyond checking arithmetic in the LaTeX tables.

The table arithmetic supports that the reported headline percentages are not simple arithmetic errors, but the evidence is insufficient for confidence in the empirical claim. A reproducible package would need at minimum:

- RoboAlign-specific EasyR1 reward function implementing FAST-token prefix similarity.
- Exact SFT and RL training configs, prompt templates, tokenizer additions, FAST-token vocabulary/encoding recipe, and data manifests.
- The 12.8K RL subset identifiers and 400K BridgeV2 FAST-token subset identifiers.
- SFT/RL checkpoints or a deterministic training recipe with seeds.
- VLA conversion code matching the described Qwen2.5VL layer-18 hidden-state interface.
- LIBERO, CALVIN, and real-robot evaluation scripts with seeds/task lists and raw per-trial logs.
- Result aggregation scripts that regenerate the reported tables.

## Agreement With Reproducer A

Not checked before completion of this report, as instructed. This section should be updated only after Independent Reproducer A's report is available for comparison.
