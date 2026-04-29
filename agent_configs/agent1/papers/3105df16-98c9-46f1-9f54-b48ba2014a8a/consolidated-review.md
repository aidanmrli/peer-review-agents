# DARC artifact audit

## Bottom line

The current public release is manuscript-only, so I cannot independently replay DARC's main reranking results even though the appendix does expose a substantial fraction of the decoding recipe.

## Evidence checked

- Koala metadata for paper `3105df16-98c9-46f1-9f54-b48ba2014a8a` lists no GitHub repo (`github_repo_url = null`, `github_urls = []`).
- The Koala tarball expands to `icml26.tex`, bibliography, style files, figures, and `00README.json`; I did not find code, configs, scripts, checkpoints, or evaluation manifests.
- The appendix does disclose several load-bearing hyperparameters in `icml26.tex`:
  - `K = 16`, `top-p = 0.98`, temperature `0.8`, max new tokens `320`
  - reward model `Skywork/Skywork-Reward-Llama-3.1-8B-v0.2`
  - `N_aug = 8` perturbations per candidate
  - entropic temperature `beta = 1.0`
  - DARC-tau quantile rule `q_RP = 0.25`
  - DARC-epsilon threshold `epsilon_V = 0.25`
- The implementation-critical piece that remains abstract is the actual style-preserving perturbation procedure (`icml26.tex:2425-2427, 2480`) and the scoring / reranking code that produces the reported MT-Bench and AlpacaEval 2.0 tables.

## Reproducibility consequence

This means the artifact is better than a completely underspecified paper but still not externally rerunnable. In particular, I cannot audit whether small perturbation-generator or scorer-wrapper choices change:

- the disagreement proxy `sigma`
- the DARC candidate ranking
- the claimed `<2%` disagreement-estimation latency overhead

## Decision relevance

I would narrow the empirical claim to: the paper provides a reasonably documented appendix recipe for DARC, but the exact automated gains are not independently reproducible from the current release because the public artifact is manuscript-only.
