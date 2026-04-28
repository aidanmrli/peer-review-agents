# DRTriton Role Findings

## Conversation triage

- Paper: `55c47c9e-cea3-4e0e-8855-342e099b5233` (`DRTriton: Large-Scale Synthetic Data Reinforcement Learning for Triton Kernel Generation`)
- Existing comments at selection time: `13` via `get_papers(limit=20)`, so the 3-comment gate was satisfied.
- Current discussion claims already covered baseline framing, test-time search dependence, verifier sparsity, and reward-gating ambiguity.
- Chosen angle: correct one repeated thread claim with direct paper evidence, then surface two cleaner paper-internal inconsistencies.

## Claim-evidence audit

- Targeted source check from Koala tarball:
  - `curl -fsSLO https://koala.science/storage/tarballs/55c47c9e-cea3-4e0e-8855-342e099b5233.tar.gz`
  - `tar -xzf ...`
  - `sed -n '706,718p' main.tex`
  - `sed -n '316,324p' main.tex`
  - `sed -n '700,705p' main.tex`
  - `rg -n "32 fundamental operators|36 fundamental operators" main.tex`
- Result 1: the GRPO ablation does specify correctness-gated speed reward:
  - `main.tex:709-716` gives `r(o) = 1 + f(t_torch / t_triton)` if the kernel is correct, else `0`.
- Result 2: the manuscript contains conflicting KernelBench headline numbers:
  - `main.tex:317` says the TTS model delivers speedups on `79%` and `60%` of Level 2/3 tasks.
  - `main.tex:323` says `92%` of KernelBench Level 2.
  - `main.tex:703-705` later reports Level 2 `92%` faster than Torch Eager and `56%` faster than `torch.compile`, and Level 3 `54%` / `34%`.
- Result 3: the operator-count description is inconsistent:
  - `main.tex:510`, `969`, `974` say the 2,026-pair SFT dataset covers `36` operators.
  - `main.tex:661` says the same 2,026-pair SFT dataset covers `32` operators.

## Literature contradiction audit

- No external literature needed for this intervention; this is a paper-internal contradiction and traceability correction.

## Logic/proof audit

- The repeated public concern that speed reward might be ungated in GRPO is contradicted by the explicit ablation formula in the paper.
- This does not fully resolve broader training-signal concerns for the main DRPO method, but it does narrow the criticism: the stronger, citable issue is inconsistent reporting, not absence of any correctness gate in the described GRPO baseline.

## Artifact-veracity audit

- No repository/code audit performed for this action.
- Evidence base is the Koala tarball only.

## Hallucination and traceability audit

- All claims above are anchored to exact `main.tex` locations extracted from the tarball.
- No unsupported extrapolation beyond those lines.

## Three citable items

1. The paper itself refutes the thread claim that the GRPO baseline leaves speed reward ungated: `main.tex:709-716` explicitly awards `1 + f(t_torch/t_triton)` only for a correct kernel, and `0` otherwise.
2. The manuscript has an unresolved KernelBench headline inconsistency: `main.tex:317` reports TTS speedups on `79%/60%` of Level 2/3 tasks, while `main.tex:323` and `703-705` report `92%` for Level 2 and `54%/34%` for Level 3 depending on denominator.
3. The SFT dataset scope is internally inconsistent: the same 2,026-pair dataset is described as covering `36` operators (`main.tex:510`, `969`, `974`) and `32` operators (`main.tex:661`).
