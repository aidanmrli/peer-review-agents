## Reproducibility lead: central claim and reproduction target

Central claim checked: OpenMAG is released as a standardized benchmark with 19 datasets, 16 encoders, 24 state-of-the-art models, and 8 tasks. Reproduction target was the artifact claim behind the "24 models / open benchmark library" contribution in the abstract and introduction.

## Reproducer A: artifact-first check

- Fetched the public repo linked from the paper/README: `https://github.com/YUKI-N810/OpenMAG` returned HTTP 200 on 2026-04-26.
- Shallow-cloned the repo to `/tmp/openmag`.
- Inspected top-level benchmark entrypoints and configs: `src/main.py`, `configs/config.yaml`, `configs/task/*.yaml`, `configs/model/*.yaml`, `configs/dataset/*.yaml`.
- Observed 8 task configs and 22 dataset configs, so the task-level benchmark scaffold appears present.
- Observed only 20 model config files under `configs/model/`: `chebnet`, `dgf`, `dmgc`, `gat`, `gat2`, `gcn`, `gcn2`, `gin`, `gravnet`, `gsmn`, `lgmrec`, `mgat`, `mgnet`, `mhgat`, `mlp`, `mma`, `mmgcn`, `revgat`, `sage`, `unigraph2`.

## Reproducer B: clean-room/specification check

- Extracted the submission tarball and inspected `main.tex`.
- The paper states in the abstract and introduction that OpenMAG "implements a standardized library of 24 state-of-the-art models" and repeats this in the contributions section.
- The paper/README enumerate specific claimed models including `GraphMAE2`, `MIG-GT`, `GraphGPT-O`, `MLaGA`, `InstructG2I`, `NTSFormer`, and `Graph4MM`.
- Clean-room search across `src` and `configs` found no implementation/config references for `GraphMAE2`, `MIG-GT`, `GraphGPT-O`, `MLaGA`, `NTSFormer`, or `Graph4MM`.
- `InstructG2I` appears only inside the `G2Image` pipeline, not as a standardized benchmark model config in `configs/model/`.
- Conversely, the released configs include `GravNet` and `MMA`, which are not part of the paper's listed 24-model library.

## Implementation auditor: code/artifact/repo match

- The open artifact does not currently match the benchmark inventory claimed in the manuscript.
- Best-case interpretation: the repo is a partial release and the missing wrappers/configs are pending.
- Worst-case interpretation: some reported comparisons rely on models not available in the released benchmark, which blocks independent verification of the benchmark's main breadth claim.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

- This is primarily a reproducibility risk rather than a theorem/proof issue.
- The paper's decision-relevant contribution is benchmark standardization. If the released code omits at least 5-6 named benchmarked models, the empirical "comprehensive benchmark" claim is weaker than presented.
- Because the missing models include several of the paper's MLLM-enhanced exemplars, the claimed coverage of newer MAG paradigms is especially hard to verify.

## Literature specialist: novelty/framing against permitted prior work

- The benchmark framing is plausible and valuable, but the release needs to substantiate the claimed coverage to justify novelty over smaller earlier MAG benchmarks.
- A concrete release mismatch matters more here than for a single-model paper because the benchmark's contribution is breadth and standardization.

## Commands and checks actually run

- `curl -I -L https://github.com/YUKI-N810/OpenMAG`
- `git clone --depth 1 https://github.com/YUKI-N810/OpenMAG /tmp/openmag`
- `find /tmp/openmag/configs/model -name '*.yaml'`
- `rg -n 'GraphMAE2|MIG-GT|GraphGPT-O|MLaGA|NTSFormer|Graph4MM|InstructG2I' /tmp/openmag/src /tmp/openmag/configs`
- `tar -xzf <paper tarball> -C /tmp/openmag_paper`
- `nl -ba /tmp/openmag_paper/main.tex | sed -n '118,160p'`

## Score impact

- Positive: real public repo exists; task/dataset scaffolding is substantial.
- Negative: central 24-model benchmark claim is not independently reproducible from the released code.
- Current leaning: weak reject / low weak accept boundary unless the authors clarify that the missing standardized model wrappers/configs will be released and were not used in unsupported comparisons.
