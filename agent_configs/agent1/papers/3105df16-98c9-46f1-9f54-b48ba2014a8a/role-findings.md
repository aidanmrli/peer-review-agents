# DARC reproducibility findings

## Central claim and reproduction target

DARC is presented as a retraining-free inference-time decoder that improves a risk-aware tradeoff by reranking a fixed candidate pool with an entropic robust value and a disagreement proxy. The smallest meaningful reproduction target is the exact automated evaluation recipe behind the main DARC results: fixed candidate pool generation, reward-model scoring, style-preserving perturbations, and the DARC-epsilon / DARC-tau selection rules.

## Paper and artifact evidence checked

- Koala paper record: `github_repo_url = null`, `github_urls = []`, tarball available.
- Koala tarball contents: manuscript-only package with `icml26.tex`, bibliography, style files, figures, and `00README.json`; no code, configs, scripts, checkpoints, or data manifests.
- `icml26.tex:2293-2308`: appendix-level decoding recipe gives the fixed pool and reward-model settings:
  - candidate pool size `K = 16`
  - generator sampling `top-p = 0.98`, temperature `0.8`, max new tokens `320`
  - reward model `Skywork/Skywork-Reward-Llama-3.1-8B-v0.2`
  - `N_aug = 8` style-preserving perturbations per candidate
  - entropic temperature `beta = 1.0`
  - DARC-tau uses per-pool `q_RP = 0.25`
  - DARC-epsilon uses `epsilon_V = 0.25`
- `icml26.tex:2425-2427, 2480`: perturbation-based disagreement is defined abstractly, but the actual perturbation generator is not released.
- `icml26.tex:816, 2658-2660`: latency claims depend on this exact implementation and say augmentation scoring adds only about `1.6%–3.2%`.

## Reproducibility result from the smallest meaningful check actually run

I unpacked the public tarball and confirmed that the release is manuscript-only. The paper text does disclose many hyperparameters in the appendix, but there is no runnable artifact for:

- candidate generation
- reward-model scoring wrapper
- style-preserving perturbation generation
- DARC selection implementation
- evaluation scripts for MT-Bench / AlpacaEval 2.0

So I can verify that the recipe is described, but I cannot independently replay the main automated results from the public artifact.

## Implementation or correctness risks

- The load-bearing disagreement proxy is operationalized through style-preserving perturbations, but the perturbation function is only described at a high level. Small implementation differences here could materially change both `sigma` and the reported latency overhead.
- The main DARC variants rely on appendix-only hyperparameters (`K`, `N_aug`, `beta`, `q_RP`, `epsilon_V`). Without code, it is hard to rule out hidden preprocessing or scorer normalization differences.
- Because there is no public script, the reported latency claim is not externally auditable even though it is deployment-relevant.

## Novelty or framing context

The paper's main novelty is inference-time disagreement-aware reranking, not artifact engineering. That makes missing code non-fatal. But the empirical claims are implementation-sensitive enough that a manuscript-only release should reduce confidence in the exact magnitude of the reported gains.

## Decision impact

My update is narrow: I would treat DARC as a promising method with a reasonably specified appendix recipe, but not as fully reproducible from the current public artifact. This lowers confidence in the exact empirical strength rather than refuting the core idea.
