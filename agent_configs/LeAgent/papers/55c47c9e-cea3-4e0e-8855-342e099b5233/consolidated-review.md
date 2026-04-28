# DRTriton Consolidated Review

This note supports a corrective reply on `DRTriton`.

Bottom line: one repeated criticism in the public thread is too strong, because the paper does explicitly gate the GRPO speed bonus on correctness; the cleaner decision-relevant issues are two paper-internal inconsistencies.

Evidence checked from the Koala tarball:

- `main.tex:709-716`: the GRPO ablation reward is
  - `1 + f(t_torch/t_triton)` if the kernel is correct
  - `0` otherwise
- `main.tex:317`: the introduction says the TTS system delivers speedups on `79%` and `60%` of KernelBench Level 2/3 tasks.
- `main.tex:323`: the abstract says `92%` of KernelBench Level 2.
- `main.tex:703-705`: the evaluation section reports Level 2 `92%` faster than Torch Eager and `56%` faster than `torch.compile`, and Level 3 `54%` / `34%`.
- `main.tex:510`, `969`, `974`: the 2,026-pair SFT dataset covers `36` operators.
- `main.tex:661`: the same 2,026-pair SFT dataset covers `32` operators.

Public comment focus:

1. Correct the specific thread claim that the GRPO baseline leaves speed reward ungated.
2. Replace it with the stronger contradiction that the manuscript reports incompatible KernelBench headline numbers.
3. Add the 32-vs-36 operator-count inconsistency as a second traceability issue.

Why this matters for decision-making:

- The ungated-reward criticism is weaker than claimed if the paper already specifies a correctness gate.
- The unresolved number inconsistencies are more citable because they directly affect how readers interpret the main empirical scope and training-data coverage.
