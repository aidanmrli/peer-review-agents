# Correctness Specialist Report

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`
Title: RoboAlign: Learning Test-Time Reasoning for Language-Action Alignment in Vision-Language-Action Models
Role: Correctness Specialist
Date: 2026-04-24

## Scope and Commands

I checked the LaTeX source, table resources, method definitions, and the linked repository snapshots under `artifacts/`.

Commands used:

```bash
sed -n '1,220p' skills/correctness-specialist.md
rg -n "RoboAlign|reward|GRPO|RL|action|token|<1|1%|Table|success|improv|benchmark|eval|real|causal|ablation" \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/main.tex \
  papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections -S
nl -ba papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/method.tex | sed -n '1,120p'
nl -ba papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/experiments.tex | sed -n '1,130p'
nl -ba papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/sections/appendix.tex | sed -n '1,90p'
nl -ba papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_LIBERO.tex
nl -ba papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_CALVIN.tex
nl -ba papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/resources/VLA_REAL.tex
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/EasyR1 rev-parse HEAD
git -C papers/69dbbf16-a716-4f44-8a1a-d1fc5b32da6b/artifacts/Isaac-GR00T rev-parse HEAD
```

Repository snapshots inspected:

- `artifacts/EasyR1`: `dd71bbd252694f5f850213eec15795b6b88d9fea`
- `artifacts/Isaac-GR00T`: `4b1dca9d88d2a0b9ea5a65aa61c82ff89f5c4f0e`

## Candidate Error 1: RL accuracy reward is mathematically ill-defined and can reward invalid action sequences

Location:

- `sections/method.tex:49-55`, especially Eq. (1).
- `sections/preliminary.tex:4-11` for FAST token construction.

Claim/definition being checked:

The paper defines the FAST-token accuracy reward as

```tex
r_a = \frac{1}{m}\max \{ i \in \{1,\dots,m\} : T^{gen}_{1:i} = T^{target}_{1:i} \}.
```

Why this is technically wrong or unsupported:

The definition omits the zero-match case. If the first generated FAST token is wrong, the set is empty and the maximum is undefined, although the paper states `r_a in [0,1]`. It also does not constrain `i <= n`, so if the generated sequence is shorter than the target sequence, prefixes `T^{gen}_{1:i}` are undefined for `i > n`. Conversely, if the generated sequence contains the entire target prefix plus extra tokens, the reward is 1.0 even though the action sequence length and decoding may be invalid. These are not presentation-only defects: this reward is the central RL objective used to claim action-aligned reasoning.

Evidence/derivation:

- For `m=5`, `T_target=[a,b,c,d,e]`, `T_gen=[x,b,c,d,e]`: the set `{i: prefix matches}` is empty, so `max` is undefined.
- For `m=5`, `T_gen=[a,b]`: the expression queries `T_gen[1:3]`, `T_gen[1:4]`, `T_gen[1:5]`, which are undefined unless an unstated convention is used.
- For `m=5`, `T_gen=[a,b,c,d,e,z]`: `r_a=1` under the stated formula, despite an extra generated action token. Because FAST uses DCT coefficients and BPE compression (`sections/preliminary.tex:9-11`), extra or missing BPE tokens are not guaranteed to correspond to benign padding.

Severity: major.

Consequence for acceptance:

The RL objective is the paper's central mechanism. The paper should either define the exact implemented reward, including empty-prefix, length, marker-token, and decoding failure handling, or demonstrate that these cases are impossible under the prompt/parser. Without that, the correctness of "action accuracy reward" is underspecified and potentially misaligned with actual low-level action correctness.

## Candidate Error 2: The "less than 1% data" claim is arithmetically narrow and causally overstated

Location:

- Abstract: `sections/abstract.tex:9`.
- Introduction: `sections/introduction.tex:31`.
- Experimental setup: `sections/experiments.tex:22-25`, `sections/experiments.tex:56`, `sections/experiments.tex:62-63`.
- Appendix compute/data details: `sections/appendix.tex:9-19`.

Claim being checked:

The paper claims that RL-based alignment uses less than 1% of the SFT data and that most performance gain comes from the RL stage despite this small data budget.

Evidence and calculations:

- The narrow unique-sample arithmetic is true: `12.8K / 2.28M = 0.00561`, i.e. 0.56%.
- However, the RL stage uses 5 samples per prompt (`sections/experiments.tex:56`), so the number of generated rollouts is at least `12.8K * 5 = 64K`, or 2.81% of the 2.28M SFT sample count if rollouts rather than unique prompts are counted.
- The paper also reports nontrivial compute: 8 H200 GPUs for 1 hour for RL (`sections/appendix.tex:9-10`). The "less than 1%" framing therefore cannot be read as less than 1% total training effort.
- The causal statement "most of the performance gain comes from the RL stage" (`sections/experiments.tex:63`) is not consistently supported by the tables. On real robot average, Qwen to `RoboAlign w/o RL` is `32.3 -> 55.2` (+22.9 absolute), while `w/o RL -> Ours` is `55.2 -> 66.7` (+11.5 absolute). There the larger gain comes before RL.

Severity: moderate.

Consequence for acceptance:

The sample-count claim is not false under a unique-prompt interpretation, but the paper uses it rhetorically to support a stronger efficiency and causal claim. That stronger claim is not established unless the authors report rollout count, token count, optimizer steps, compute-normalized comparison, and a consistent per-benchmark decomposition of SFT vs RL gains.

## Candidate Error 3: Baseline and ablation comparisons are confounded by unequal data and task definitions

Location:

- Baseline setup: `sections/experiments.tex:29-31`.
- LIBERO results: `resources/VLA_LIBERO.tex:23-27`.
- Alignment strategy comparison: `sections/experiments.tex:85-89`, `sections/appendix.tex:23-24`, `resources/VLA_LIBERO_change_alignment.tex:12-15`.
- SFT-vs-RL comparison: `sections/experiments.tex:93-98`, `sections/appendix.tex:26`, `resources/SFT_vs_RL_LIBERO.tex:12-14`.

Claim being checked:

The paper states that direct low-level action RL alignment is more effective than language-based RL, visual-trajectory RL, and SFT-based alignment, and that these results demonstrate an advantage of direct alignment with low-level actions.

Why this is technically unsupported:

The ablations do not isolate the alignment target as the only variable.

- In Table `VLA_LIBERO`, `RoboAlign w/o RL` uses 2.28M samples, while Language-Only SFT and Action-Only SFT each use 1.88M (`resources/VLA_LIBERO.tex:24-27`). This makes the main baseline comparison a mixture of data-composition and data-volume changes.
- The alternative alignment strategies use different supervision formats and different data volumes. Appendix `sections/appendix.tex:23-24` says language-based RL uses converted movement MCQA, visual trajectory RL uses ShareRobot and is limited to 6K samples, while RoboAlign uses a 12.8K BridgeV2 FAST-token subset. Thus Table `VLA_LIBERO_change_alignment` does not isolate "low-level action target" from dataset source, sample count, answer format, and reward implementation.
- The SFT-based ECoT comparison uses one epoch of SFT on 12.8K samples on top of the same SFT model (`sections/appendix.tex:26`), but this is not an RL-vs-SFT comparison with matched optimization pressure, generated rollouts, or objective token budget.

Evidence from table arithmetic:

- In the alignment table, visual-based RL is close to action-based RL overall: 85.1 vs 86.8 average (`resources/VLA_LIBERO_change_alignment.tex:14-15`). The advantage is concentrated in Long (64.6 vs 70.0) while action-based RL is lower than visual-based RL in Goal (87.2 vs 87.8) and Object (96.0 vs 95.6 is only +0.4).
- The paper's conclusion that the table "further demonstrates the advantage of direct alignment with low-level actions" (`sections/experiments.tex:89`) is stronger than this confounded experiment permits.

Severity: major.

Consequence for acceptance:

The ablation evidence supports "this full training recipe performs best in these reported runs," but it does not establish that the low-level action target is the decisive causal factor. This materially weakens the central mechanism claim.

## Candidate Error 4: Real-robot evaluation description is internally inconsistent and the RL gain is not statistically established

Location:

- Real-robot prose: `sections/experiments.tex:79`.
- Real-robot table/caption: `resources/VLA_REAL.tex:1-13`.

Claim being checked:

The paper claims real-robot gains extend beyond simulation and reports "evaluated over 96 trials per task" in the table caption.

Why this is technically wrong or unsupported:

The real-world trial accounting is inconsistent. The prose says four tasks, "24 trials per object, totaling 96 trials per task" (`sections/experiments.tex:79`). Table `VLA_REAL` has four task columns and percentages that are multiples of 1/24: 16.7=4/24, 70.8=17/24, 87.5=21/24, 58.3=14/24, 37.5=9/24, 50.0=12/24. This implies 24 trials per task and 96 total trials per model, not 96 trials per task. The caption's "96 trials per task" is therefore inconsistent with the reported percentages.

The real-robot RL improvement over `RoboAlign w/o RL` is also small-sample and not supported by uncertainty estimates. Converting the table to counts:

- Qwen: approximately `31/96`.
- RoboAlign w/o RL: approximately `53/96`.
- RoboAlign: approximately `64/96`.

An approximate pooled two-proportion z-test for RoboAlign vs RoboAlign w/o RL gives `z=1.63`, two-sided `p=0.104`. The normal 95% half-widths are about 9-10 percentage points for these proportions. Thus the reported +11.5 absolute average improvement over the no-RL variant is not clearly statistically established without repeated seeds, confidence intervals, or task-level paired analysis.

Severity: moderate.

Consequence for acceptance:

The real-robot comparison to the original Qwen baseline appears large, but the incremental claim that the RL stage itself "consistently improves performance even in real-robot settings" is not adequately supported by this small and inconsistently described evaluation.

## Candidate Error 5: Reported headline improvements depend on the weakest baseline rather than the SFT baseline named in the abstract

Location:

- Abstract: `sections/abstract.tex:9`.
- Introduction: `sections/introduction.tex:31`.
- LIBERO table: `resources/VLA_LIBERO.tex:23-27`.
- CALVIN table: `resources/VLA_CALVIN.tex:12-16`.
- Real table: `resources/VLA_REAL.tex:11-13`.

Claim being checked:

The abstract says RoboAlign achieves improvements of 17.5%, 18.9%, and 106.6% "over SFT baselines" on LIBERO, CALVIN, and real-world environments.

Evidence and calculations:

The 17.5%, 18.9%, and 106.6% figures match comparisons against the original Qwen model for LIBERO/CALVIN/real, not consistently against the strongest SFT baseline:

```text
LIBERO: (86.8 - 73.9) / 73.9 = 17.46%
CALVIN average length: (2.57 - 2.16) / 2.16 = 18.98%
Real: (66.7 - 32.3) / 32.3 = 106.50%
```

But Table `VLA_LIBERO` includes SFT baselines with averages 79.6, 81.5, and 78.7. Against the strongest SFT baseline, the LIBERO relative gain is `(86.8 - 81.5) / 81.5 = 6.50%`, not 17.5%. Against `RoboAlign w/o RL`, it is 10.29%. For real robot, the available no-RL SFT baseline is 55.2, so the RL-stage relative gain is `(66.7 - 55.2) / 55.2 = 20.83%`, not 106.6%.

Severity: moderate.

Consequence for acceptance:

The headline improvement statement is imprecise and likely misleading. It should specify the exact comparator for each percentage. The central empirical gain remains positive, but the headline magnitude is overstated if read as improvement over the best or directly corresponding SFT baseline.

## Candidate Error 6: Claims about improved representations and MLLM capabilities are over-scoped relative to the evaluation design

Location:

- Representation analysis: `sections/experiments.tex:101-104`, `sections/appendix.tex:28-30`, `resources/knn.tex:10-12`.
- MLLM benchmark claim: `sections/experiments.tex:106-108`, `resources/mllm_bench.tex`.
- Conclusion: `sections/conclusion.tex:4-7`.

Claim being checked:

The paper claims RL alignment "significantly sharpens" fine-grained state representations and improves general MLLM capabilities.

Why this is unsupported:

The KNN representation analysis uses 20 training trajectories from a single LIBERO long-horizon task (`sections/appendix.tex:28-30`). The labels are produced by DTW over robot states, and the classifier sees hidden representations from the same task distribution. This is insufficient to conclude a general mechanism for VLA performance improvement across LIBERO, CALVIN, and real robots. It is at most a task-local diagnostic.

The MLLM benchmark table also does not support the broad claim that RL improves general MLLM capabilities. On MMStar, RoboAlign (62.80) is only +0.33 over RoboAlign w/o RL (62.47) and lower than GPT-4o (65.10) and VeBrain (61.90 is lower, but close). For spatial benchmarks, RoboAlign remains below RoboBrain2.0 on RoboSpatial (50.86 vs 54.23) and Where2Place (54.49 vs 63.59). Thus the statement in `sections/experiments.tex:108` that the method "consistently outperforms specialized embodied reasoning models" is only true for some columns, not consistently across the reported embodied/spatial benchmarks.

Severity: moderate.

Consequence for acceptance:

The representation and general-capability conclusions should be narrowed. These analyses are useful but do not establish a general mechanism or state-of-the-art embodied reasoning performance across the full benchmark set.

## Candidate Error 7: Artifact inspection does not expose the paper-specific reward, data pipeline, or evaluation scripts needed to validate implementation correctness

Location:

- Implementation claims: `sections/experiments.tex:52-56`, `sections/appendix.tex:12-19`.
- Artifact repositories: `artifacts/EasyR1`, `artifacts/Isaac-GR00T`.

Finding:

The linked repositories are generic EasyR1 and Isaac-GR00T snapshots. Searching the provided artifacts for paper-specific identifiers and reward/pipeline terms did not reveal a RoboAlign implementation, BridgeV2 FAST-token construction script, custom reward function, LIBERO/CALVIN evaluation script, or exact configs for the reported runs. EasyR1 contains a generic custom reward loading mechanism, but no provided paper-specific reward file was visible in the artifact tree inspected.

Evidence:

- `git rev-parse HEAD` confirms generic snapshots at the commits listed above.
- `find artifacts -maxdepth 4 ... | rg -i "(reward|roboalign|bridge|libero|calvin|fast|action|grpo|train|eval|config)"` found generic framework files and LaTeX resources but no RoboAlign-specific training/evaluation configuration.
- The paper states "we use the EasyR1 repository" and "implementation refers to the GR00T-N1.5 codebase," but does not provide the exact modifications/configs necessary to verify whether the implemented reward matches Eq. (1), how malformed outputs are parsed, or how action-token outputs are fed into VLA training.

Severity: major for reproducibility and implementation correctness; moderate as a pure paper-logic error.

Consequence for acceptance:

The implementation-dependent claims cannot be independently checked from the supplied artifacts. This matters because the main correctness concerns above depend precisely on parser/reward/evaluation details that might differ from the paper's mathematical description.

## Arithmetic Checks That Passed

The row averages in the main tables are mostly arithmetically consistent after rounding:

- LIBERO RoboAlign: `(93.8 + 96.0 + 87.2 + 70.0)/4 = 86.75`, reported 86.8.
- LIBERO w/o RL: `(92.8 + 97.4 + 59.0 + 65.6)/4 = 78.7`, reported 78.7.
- Real RoboAlign: `(87.5 + 58.3 + 70.8 + 50.0)/4 = 66.65`, reported 66.7.
- Qwen3 RoboAlign: `(95.6 + 99.6 + 95.2 + 78.6)/4 = 92.25`, reported 92.5 if rounded to one decimal should be 92.3, so the Qwen3 average appears to be a minor arithmetic or source-table inconsistency.

CALVIN average sequence length appears consistent with the standard sum of success probabilities over sequence lengths:

- Qwen: `(77.8 + 55.0 + 38.6 + 26.6 + 18.1)/100 = 2.161`, reported 2.16.
- RoboAlign: `(87.6 + 67.2 + 47.1 + 32.8 + 22.2)/100 = 2.569`, reported 2.57.

## Final Synthesis and Score Impact

The paper presents positive empirical results, and the main table arithmetic mostly checks out. However, the central correctness weakness is that the RL reward, which is the core claimed mechanism, is mathematically underspecified and potentially misaligned with valid FAST action sequences. The causal interpretation of the ablations is also too strong because alignment target, data source, sample count, and objective format are confounded. Real-world claims are weakened by inconsistent trial accounting and missing uncertainty estimates. Finally, artifact inspection did not expose the paper-specific reward/parser/configs needed to resolve these issues.

Score impact: these issues are decision-relevant and should materially reduce confidence. I would not treat the claimed mechanism as established without corrected reward definition, exact implementation release, and cleaner matched ablations. The empirical performance signal remains useful, but the correctness evidence supports at most a cautious weak-accept/weak-reject boundary depending on other roles' reproducibility findings, not a strong accept.
