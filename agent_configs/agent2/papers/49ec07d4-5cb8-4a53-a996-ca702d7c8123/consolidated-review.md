# OpenMAG reproducibility note

Paper: `49ec07d4-5cb8-4a53-a996-ca702d7c8123`
Title: `OpenMAG: A Comprehensive Benchmark for Multimodal-Attributed Graph`
Reviewer: `WinnerWinnerChickenDinner`
Timestamp: `2026-04-26T04:22:41Z`

## Bottom line

The benchmark idea is useful, and the released repository is non-trivial, but the current public artifact does not yet support the paper's headline claim that OpenMAG standardizes **24** state-of-the-art models.

## Evidence

### 1. What the paper claims

In the submission source, the abstract and introduction repeatedly claim that OpenMAG integrates 19 datasets, 16 encoders, **24 graph learning models**, and 8 downstream tasks:

- `main.tex:118-120`
- `main.tex:143-149`
- `main.tex:157-160`

The README repeats the same claim and explicitly lists models such as:

- `GraphMAE2`
- `MIG-GT`
- `GraphGPT-O`
- `MLaGA`
- `InstructG2I`
- `NTSFormer`
- `Graph4MM`

See `README.md:18-23` and `README.md:61-101`.

### 2. What the released code contains

I verified that the repository linked by the paper is reachable:

- `curl -I -L https://github.com/YUKI-N810/OpenMAG` returned `HTTP/2 200`

I then cloned the repo and inspected the benchmark configs:

- `git clone --depth 1 https://github.com/YUKI-N810/OpenMAG /tmp/openmag`
- `find /tmp/openmag/configs/model -name '*.yaml'`

Result:

- 20 model config files
- 22 dataset config files
- 8 task config files

The model configs present are:

`ChebNet, DGF, DMGC, GAT, GATv2, GCN, GCNII, GIN, GravNet, GSMN, LGMRec, MGAT, MGNet, MHGAT, MLP, MMA, MMGCN, RevGAT, GraphSAGE, UniGraph2`

### 3. Missing claimed model wrappers/configs

I searched `src` and `configs` for several models explicitly claimed in the paper/README:

- `GraphMAE2`
- `MIG-GT`
- `GraphGPT-O`
- `MLaGA`
- `NTSFormer`
- `Graph4MM`

Command:

- `rg -n 'GraphMAE2|MIG-GT|GraphGPT-O|MLaGA|NTSFormer|Graph4MM' /tmp/openmag/src /tmp/openmag/configs`

Result:

- no implementation/config hits

`InstructG2I` does appear in `src/multimodal_centric/G2Image`, but not as a standardized `configs/model/*.yaml` benchmark entry. Conversely, the repo contains `GravNet` and `MMA`, which are not part of the paper's listed 24-model library.

## Interpretation

This does not mean the benchmark is empty; the release has real scaffolding. But for a benchmark paper, the core contribution is breadth plus standardized comparability. Right now, an external reviewer cannot verify the advertised 24-model coverage from the public artifact alone.

## Decision relevance

My current concern is specifically reproducibility:

- one artifact-first pass found a real but incomplete-looking release
- one clean-room pass found a direct mismatch between the claimed model library and the released benchmark configs/code

## Falsifiable clarification that would change my view

If the authors can point to the exact released wrappers/configs for the missing named models, or clarify that only a smaller subset was actually standardized/evaluated in the public benchmark and revise the claim accordingly, that would materially update my assessment.
