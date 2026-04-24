# Consolidated Review Evidence

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`
Title: RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models
Agent: `agent2`
Date: 2026-04-24

## Executive Conclusion

The paper presents an interesting and plausible idea: use R1-style RL to align explicit reasoning traces with low-level FAST action-token prediction, then convert the resulting MLLM into a GR00T-style VLA. However, the central empirical claims are not independently reproducible from the supplied artifacts. Two independent reproducers recovered only table arithmetic; neither could reproduce the LIBERO, CALVIN, or real-robot results, because the released repositories are generic EasyR1 and Isaac-GR00T snapshots rather than a runnable RoboAlign release.

This is decision-relevant. The headline performance claims drive the acceptance case, yet the artifacts omit the exact reward implementation, training data construction, BridgeV2 FAST-token subsets, tokenizer changes, SFT/RL configs, checkpoints, VLA conversion code, and benchmark evaluation scripts needed to verify them.

Recommended verdict range if no additional artifacts become available: 3.5 to 4.5.

## Central Claim Tested

The main claim tested was that RoboAlign's SFT+RL alignment, using less than 1% additional BridgeV2 FAST-token data, improves VLA performance on LIBERO, CALVIN, and real-robot manipulation tasks with headline gains of 17.5%, 18.9%, and 106.6%, while preserving or improving MLLM reasoning ability.

Key paper locations:

- `sections/method.tex:34-39`: vocabulary extension with `<ACTION_START>`, `<ACTION_END>`, and 2K FAST tokens; BridgeV2 FAST-token QA construction.
- `sections/method.tex:41-57`: RL stage and reward `r=(r_f+r_a)/2`, with `r_a` as FAST-token prefix similarity.
- `sections/experiments.tex:19-25`: GR00T-style VLA conversion and SFT/RL data sizes.
- `sections/experiments.tex:52-56`: Qwen2.5VL-7B-Ins, EasyR1, rollout batch 512, update batch 128, 5 samples per prompt, LR `1e-6`.
- `sections/appendix.tex:9-15`: 8 H200 MLLM training, 2 A100 VLA training, GR00T-N1.5, Qwen2.5VL hidden states from layer 18, benchmark-specific steps.
- `resources/VLA_LIBERO.tex:23-27`, `resources/VLA_CALVIN.tex:12-16`, `resources/VLA_REAL.tex:11-13`: main benchmark tables.

## Role-By-Role Findings

### Independent Reproducer A

Reproducer A checked the paper source, main result tables, the linked EasyR1 and Isaac-GR00T repositories, and the local Python environment. The role reproduced arithmetic consistency but no empirical result:

- `12.8K / 2.28M = 0.561%`, supporting the narrow less-than-1% unique-prompt arithmetic.
- CALVIN `2.57` versus `2.16` gives `18.98%`; real robot `66.7` versus `32.3` gives `106.50%`.
- LIBERO `86.8` versus raw Qwen `73.9` gives `17.46%`, but `86.8` versus RoboAlign w/o RL `78.7` gives only `10.29%`.
- No RoboAlign SFT command, RL command, BridgeV2 subset, custom reward, checkpoint, VLA conversion code, or benchmark evaluation script was present.
- Local environment lacked PyTorch, Ray, EasyR1/verl, GR00T, and visible GPU support, but the larger blocker was missing paper-specific artifacts.

Outcome: blocked empirical reproduction; partial arithmetic reproduction only.

### Independent Reproducer B

Reproducer B independently traced table consistency and code sufficiency. This role did not read Reproducer A before writing its report and reached the same substantive conclusion:

- Headline percentages are reconstructible only with specific denominator choices, and the phrase "over SFT baselines" is imprecise.
- The EasyR1 clone supports custom rewards in principle but contains no RoboAlign FAST-token prefix reward, no BridgeV2 FAST-token data recipe, and no RoboAlign launch config.
- The Isaac-GR00T clone is an upstream GR00T N1.7-oriented repository. It does not instantiate the paper's Qwen2.5VL-7B layer-18 GR00T-N1.5 setup.
- No raw metrics, checkpoints, prediction logs, CALVIN path, real-robot logs, or KNN representation features were found.

Outcome: blocked empirical reproduction; partial table arithmetic consistency only.

### Implementation Auditor

The auditor inspected:

- EasyR1 commit `dd71bbd252694f5f850213eec15795b6b88d9fea`.
- Isaac-GR00T commit `4b1dca9d88d2a0b9ea5a65aa61c82ff89f5c4f0e`.
- Paper source, result tables, EasyR1 examples/reward functions/configs, and GR00T configs/evaluation scripts.

Paper-to-code matches were generic only: EasyR1 supports GRPO and custom reward loading, and GR00T supports frozen-backbone action-head training. Paper-to-code discrepancies were severe:

- No RoboAlign-specific EasyR1 reward implementing format reward plus FAST-token prefix accuracy.
- No SFT data construction or FAST-token vocabulary extension code.
- No RoboAlign EasyR1 training configs for Qwen2.5VL-7B, BridgeV2, and 12.8K RL data.
- GR00T code does not match Qwen2.5VL-7B layer-18 VLA conversion; the inspected config is Qwen3/Cosmos-oriented with `select_layer=12`.
- LIBERO scripts are generic, with different step counts and public NVIDIA checkpoints.
- No CALVIN implementation/evaluation path.
- No real-robot dataset, evaluation script, trial log, or success labels.
- No checkpoints or model cards for reported variants.

Outcome: high-severity implementation reproducibility failure.

### Correctness Specialist

The correctness review found several decision-relevant issues:

- The reward at `sections/method.tex:49-55` is mathematically underspecified. If the first token mismatches, the max set is empty; if generation is shorter than target, some prefixes are undefined; if generation includes the full target plus extra tokens, the written formula still gives reward 1.0. This matters because the reward is the central mechanism.
- The less-than-1% data claim is true for unique prompts but overstates training efficiency if rollout count, token count, compute, or 8 H200 GPU-hours are considered.
- The claim that "most" gain comes from RL is not consistently supported. In the real-robot table, raw Qwen to no-RL SFT is `32.3 -> 55.2` (+22.9), while no-RL to RL is `55.2 -> 66.7` (+11.5).
- Ablations are confounded by data source, sample count, objective format, and supervision type. For example, visual-based RL uses only 6K ShareRobot samples, while action-based RL uses 12.8K BridgeV2 FAST-token samples.
- Real-robot trial accounting is inconsistent: the table caption says 96 trials per task, but table values are multiples of 1/24 and imply 24 trials per task, 96 total trials per model. The no-RL to RL improvement is not accompanied by confidence intervals or paired analysis.
- KNN representation analysis uses only 20 training trajectories from one LIBERO long-horizon task, limiting the mechanism claim.

Outcome: major correctness and interpretation risks.

### Literature Specialist

The literature review found the contribution plausible but narrower than framed:

- FAST/OpenVLA/RT-2-style work already establishes low-level/discrete action tokenization.
- GR00T, pi0/pi0.5, Octo, Diffusion-VLA, DexVLA, and CogAct cover frozen or partially frozen VLM backbones with action heads.
- ECoT, CoT-VLA, ThinkAct, Robot-R1, ManipLVM-R1, RoboBrain, RoboVQA, and related work already explore embodied reasoning and robot-oriented reasoning supervision.
- DeepSeek-R1, GRPO, EasyR1, and multimodal R1-style work establish the post-training pattern.

The narrow novelty is using GRPO/R1-style RL to optimize explicit reasoning traces against low-level FAST-token prediction accuracy, then testing whether that improves downstream GR00T-style VLA performance. That is a useful target choice but not a new architecture, action representation, or RL algorithm.

Outcome: borderline-to-weak-accept novelty if empirical evidence holds; insufficient to offset unreproducible central results.

## Evidence Table

| Evidence | Source or command | Outcome |
| --- | --- | --- |
| Less-than-1% unique-prompt arithmetic | `12800 / 2280000 * 100` | `0.561%`, arithmetically consistent |
| LIBERO headline denominator | `resources/VLA_LIBERO.tex:23-27` | `86.8` vs `73.9` gives `17.46%`; vs no-RL `78.7` gives `10.29%` |
| CALVIN headline denominator | `resources/VLA_CALVIN.tex:12-16` | `2.57` vs `2.16` gives `18.98%`; vs no-RL `1.89` gives `35.98%` |
| Real headline denominator | `resources/VLA_REAL.tex:11-13` | `66.7` vs `32.3` gives `106.50%`; vs no-RL `55.2` gives `20.83%` |
| RL reward definition | `sections/method.tex:49-55` | Empty-prefix and length cases undefined; extra-token case not penalized as written |
| VLA conversion claim | `sections/appendix.tex:13-15` | Requires GR00T-N1.5 and Qwen2.5VL layer 18; not found in released GR00T clone |
| EasyR1 implementation | `rg -n "RoboAlign|BridgeV2|FAST|ACTION|prefix|accuracy reward" artifacts/EasyR1` | No RoboAlign-specific reward/config/data pipeline |
| GR00T implementation | `rg -n "RoboAlign|CALVIN|Qwen2.5|layer.*18|GR00T-N1.5" artifacts/Isaac-GR00T` | Generic GR00T N1.7-oriented code; no RoboAlign conversion/eval |
| Checkpoints/logs | `find artifacts -type f \( -name '*.safetensors' -o -name '*.bin' -o -name '*.pt' -o -name '*.ckpt' -o -name '*metrics*' -o -name '*result*' \)` | No RoboAlign model checkpoints, logs, or table reproduction files |
| KNN mechanism evidence | `sections/appendix.tex:28-30`, `resources/knn.tex:10-12` | 20 trajectories from one training task; not a general mechanism validation |
| MLLM benchmark framing | `resources/mllm_bench.tex:16-24` | RoboAlign is not best on RoboSpatial or Where2Place; broad "consistently outperforms" wording is too strong |

## Commands and Environment Details

Representative commands used by the team:

```bash
rg -n "RoboAlign|LIBERO|CALVIN|SFT|reinforcement|RL|EasyR1|GR00T|Isaac" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/main.tex \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections -S

rg -n "RoboAlign|roboalign|BridgeV2|FAST|action_start|think|LIBERO|CALVIN|Qwen2.5|Qwen3|GRPO|reward|accuracy|diffusion|hidden|18" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T

git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 rev-parse HEAD
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T rev-parse HEAD

find papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts -type f \
  \( -name '*.safetensors' -o -name '*.bin' -o -name '*.pt' -o -name '*.pth' -o -name '*.ckpt' -o -name '*metrics*' -o -name '*result*' \) | sort
```

Local environment observed by Reproducer A:

```text
Python 3.12.12
torch: missing
transformers: missing
datasets: missing
ray: missing
verl: missing
gr00t: missing
No visible GPU listing from nvidia-smi in the shell.
```

This local environment limitation is secondary; the main blocker is absence of paper-specific reproducibility artifacts.

## Reproduction Outcome

Strong reproducibility: no.

Partial reproducibility: yes, for internal table arithmetic and data-volume arithmetic only.

Weak reproducibility: yes, for the central empirical claims. Neither independent reproducer could recover the main benchmark results or execute a faithful RoboAlign pipeline from the available artifacts.

Contradicted or materially weakened claims:

- "17.5%, 18.9%, 106.6% over SFT baselines" is comparator-dependent and imprecise.
- "Most gain comes from RL" is not consistently supported across tables.
- The reward definition is underspecified and may not reflect valid FAST-token action correctness.
- Broad claims about representation sharpening and MLLM benchmark superiority are stronger than the evidence supports.

## Draft Public Comment

Bottom line: I would not credit the headline VLA gains as reproducible from the supplied artifacts; both independent reproduction passes recovered only table arithmetic, not the RoboAlign empirical pipeline.

Our internal reproduction team checked the LaTeX source, result tables, and the two linked repositories. The narrow arithmetic mostly works: `12.8K / 2.28M = 0.56%`, CALVIN `2.57` vs `2.16` gives `18.98%`, real robot `66.7` vs `32.3` gives `106.50%`, and table averages are generally rounded consistently. However, the LIBERO `17.5%` headline matches `86.8` vs raw Qwen `73.9`, not the directly corresponding `RoboAlign w/o RL` SFT row `78.7`, where the gain is `10.29%`.

The substantive reproducibility failure is in the artifacts. The EasyR1 snapshot contains generic GRPO examples and custom-reward loading, but I found no RoboAlign FAST-token prefix-similarity reward, BridgeV2 12.8K RL subset, tokenizer/vocabulary extension, SFT/RL launch config, checkpoint, or training log. The Isaac-GR00T snapshot is likewise a generic upstream GR00T repo; it does not instantiate the paper's stated GR00T-N1.5/Qwen2.5VL-7B layer-18 VLA conversion, and I found no CALVIN or real-robot reproduction path for the reported tables.

There is also a correctness issue in the written reward at `method.tex:49-55`: if the first generated FAST token mismatches, the max set is empty; if the generated sequence is shorter than the target, some prefixes are undefined; and if it emits the target prefix plus extra action tokens, the stated reward can still reach 1.0. Since this reward is the core mechanism, the exact parser/reward implementation is not a detail; it is necessary evidence.

My conclusion is that the idea is literature-plausible, but the current release supports only an internally consistent table, not an independently reproducible robotics result. The paper should provide the exact RoboAlign reward implementation, BridgeV2/FAST data manifests, tokenizer changes, SFT/RL configs, checkpoints, VLA conversion code, and LIBERO/CALVIN/real-robot evaluation scripts before the central claims receive high confidence.

## Score Impact

I would assign a weak-reject range at this stage, approximately 3.5 to 4.5. The paper is topical and the target choice is useful, but reproducibility is the dominant axis for this agent. A robotics paper whose acceptance case depends on large benchmark gains must make those gains independently auditable; here the reported numbers are not reproducible from the available artifacts.
