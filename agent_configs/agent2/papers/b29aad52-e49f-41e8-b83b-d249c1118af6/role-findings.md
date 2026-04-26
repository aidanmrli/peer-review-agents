# Role Findings for `b29aad52-e49f-41e8-b83b-d249c1118af6`

## Reproducibility lead: central claim and reproduction target
- Central claim checked: the released artifacts are sufficient to audit or rerun the reported SFT + GRPO pipeline and the main ORDerly results for RetroReasoner.
- Outcome: partial paper-level transparency, but not reproduction-grade. The tarball contains LaTeX, prompts, formulas, and hyperparameter tables; it does not contain code, checkpoints, dataset instance lists, or evaluation scripts.
- Commands/checks run:
  - `tar -tzf .../source.tar.gz`
  - `find .../artifacts/source -maxdepth 2 -type f`
  - `rg -n "code|public|available|randomly selected|Qwen3|GPT-oss|ORDerly|round-trip|cache" .../artifacts/source`

## Reproducer A: artifact-first check
- Koala metadata shows `github_repo_url: null` and `github_urls: []`.
- The source bundle is manuscript-only: `main.tex`, section files, figures, tables, style files, and prompts are present; no training code, configs as executable files, scripts, checkpoints, logs, or dataset manifests were shipped.
- The related-work comparison table says RetroReasoner's supporting resources are `"(will be publicly available.)"` in [table_tex/molecular_reasoning_LLMs_comparison.tex](./artifacts/source/table_tex/molecular_reasoning_LLMs_comparison.tex).
- Reproduction consequence: I can audit the intended workflow, but I cannot rerun the experiments or verify the reported numbers from the official artifacts alone.

## Reproducer B: clean-room/specification check
- The appendix gives useful specifications:
  - SFT/RL hyperparameters in [table_tex/hyperparameters_SFT.tex](./artifacts/source/table_tex/hyperparameters_SFT.tex) and [table_tex/hyperparameters_RL.tex](./artifacts/source/table_tex/hyperparameters_RL.tex)
  - prompts in [contents/99_06_prompts.tex](./artifacts/source/contents/99_06_prompts.tex)
  - SyntheticRetro generation setup in [contents/99_01_details_of_SyntheticRetro.tex](./artifacts/source/contents/99_01_details_of_SyntheticRetro.tex)
  - metric formulas and subset construction in [contents/99_05_details_of_experiment.tex](./artifacts/source/contents/99_05_details_of_experiment.tex)
- Clean-room blocker 1: the evaluation sets are randomly sampled but no seed or released instance IDs are provided. This affects:
  - the 500-example in-distribution set
  - the 100-example rare-template set
  - the 100-example rare-token set
- Clean-room blocker 2: the round-trip verifier `f_phi` is central to both RL reward and evaluation metrics, but the trained verifier checkpoints and exact data split are not released.
- Clean-room blocker 3: the method depends on external model components and preprocessing tools (`GPT-oss-20B`, `Qwen3-*`, `LocalMapper`, vLLM) but no runnable orchestration code is provided.

## Implementation auditor: code/artifact/repo match
- The manuscript describes a nontrivial implementation:
  - eight H100s for SFT
  - eight H100s for RL
  - three rollout workers plus a dedicated round-trip worker
  - a shared cache for verifier calls
  - asynchronous SyntheticRetro generation with eight GPUs and multiple vLLM consumers
- Those implementation details appear in prose only; no executable implementation or config files were shipped.
- The release mismatch matters because several headline claims depend on infrastructure, not only formulas:
  - large-scale SyntheticRetro generation
  - cached round-trip reward computation
  - distributed RL training

## Correctness specialist: methods, metrics, proofs, or conclusion risks
- The strongest evaluation risk is that the same round-trip model family underwrites both optimization and the main feasibility-style metrics (`Round-trip@1`, `Round-trip@100`, `Feasible Ratio`, `Template Diversity`).
- The appendix describes the round-trip model but does not release the verifier, its checkpoint, or the exact held-out split used for the published metric tables.
- There are commented-out lines in [contents/99_04_details_of_roundtrip.tex](./artifacts/source/contents/99_04_details_of_roundtrip.tex) indicating the authors considered combining ORDerly train and test for verifier training before selecting a validation subset. Even though those lines are commented out, they make the final split protocol important enough that it should be stated explicitly and reproducibly.
- Score impact: this lowers confidence in the reported absolute gains because an independent reader cannot reconstruct the exact reward/evaluation oracle.

## Literature specialist: novelty/framing against permitted prior work
- My comment will not dispute the paper's core novelty claim directly.
- The decision-relevant framing point is narrower: the paper offers substantially more transparency than a no-artifact submission, but still falls short of reproducibility for a method whose contribution depends on a generated reasoning dataset, a custom verifier, and custom sampled evaluation subsets.
- Missing code, checkpoints, and subset manifests should therefore be treated as a material confidence discount rather than a fatal originality objection.
