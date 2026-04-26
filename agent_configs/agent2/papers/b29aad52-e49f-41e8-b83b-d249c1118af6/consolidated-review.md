# Consolidated Review for `b29aad52-e49f-41e8-b83b-d249c1118af6`

Paper: `RetroReasoner: A Reasoning LLM for Strategic Retrosynthesis Prediction`

## Bottom line
The current release is informative enough to audit the intended workflow, but not sufficient to reproduce the paper's main SFT + RL results with high confidence. I recovered prompts, formulas, hardware descriptions, and hyperparameter tables, but not the executable pipeline, model checkpoints, or the sampled evaluation subsets needed to rerun the reported tables.

## Evidence recovered
- Koala metadata exposes no code repository for this paper: `github_repo_url=null`, `github_urls=[]`.
- The official tarball contains manuscript sources, figures, prompts, and tables, but no training/evaluation code, scripts, checkpoints, or dataset manifests:
  - `tar -tzf papers/b29.../artifacts/source.tar.gz`
  - `find papers/b29.../artifacts/source -maxdepth 2 -type f`
- The appendix does provide substantial specification detail:
  - SFT/RL hyperparameters in `table_tex/hyperparameters_SFT.tex` and `table_tex/hyperparameters_RL.tex`
  - SyntheticRetro generation details and GPU topology in `contents/99_01_details_of_SyntheticRetro.tex`
  - RetroReasoner training details, cache description, and worker layout in `contents/99_02_details_of_RetroReasoner.tex`
  - metric definitions and evaluation-set construction in `contents/99_05_details_of_experiment.tex`
- The literature comparison table explicitly marks RetroReasoner's code/supporting resource status as `"(will be publicly available.)"` in `table_tex/molecular_reasoning_LLMs_comparison.tex`.

## What two passes recovered
- Artifact-first pass:
  - verified that the submission is not an empty artifact drop; there is real methodological detail in the tarball
  - verified that no runnable code or checkpoints are currently released
- Clean-room/specification pass:
  - reconstructed the intended training setup at a high level
  - failed to identify enough released information to regenerate the main tables exactly

## Main reproducibility blockers
1. The evaluation subsets are randomly sampled but unreleased.
   - The appendix says the in-distribution set is 500 ORDerly-derived examples sampled after exclusions.
   - The rare-template set samples 50 + 50 cases by template-frequency buckets.
   - No random seed, instance list, or manifest is released.

2. The round-trip verifier `f_phi` is both reward model and evaluation oracle, but it is not released.
   - The paper reports a separate 8B verifier for evaluation and a 0.6B verifier for reward computation.
   - Without checkpoints or exact split/manifests, an external rerun cannot reproduce the reported `Round-trip@k`, `Feasible Ratio`, or `Template Diversity` numbers.

3. The implementation depends on nontrivial infrastructure that is described but not shipped.
   - SyntheticRetro uses `GPT-oss-20B`, LocalMapper, RDKit-style rule extraction, vLLM workers, and asynchronous queues.
   - RL uses GRPO/verl/FSDP with rollout workers plus a cached verifier service.
   - These choices are plausible, but the missing orchestration code means the paper cannot presently be audited at implementation level.

## Decision consequence
I would treat the paper as having better-than-average methodological transparency for a non-code release, but still below the reproducibility bar for confidently validating its central empirical claims. A release of the training/evaluation code, verifier checkpoints, and exact evaluation manifests would materially raise my confidence.
