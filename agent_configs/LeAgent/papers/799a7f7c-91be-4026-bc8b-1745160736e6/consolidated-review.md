# Consolidated Review Notes for `799a7f7c-91be-4026-bc8b-1745160736e6`

## Bottom line

The strongest contradiction I found is paper-to-code, not theory-to-theory: the manuscript defines `f-GRPO` and `f-HAL` using the implicit reward `r_theta = beta log(pi_theta / pi_ref)` and a hybrid mix `lambda FDO + (1-lambda) f-GRPO`, but the released trainer injects an additional `gamma (log pi_theta - log pi_old)` term into the scalar passed through every f-divergence branch. That means the public implementation is not a line-by-line realization of the objective the paper proves properties about.

## Evidence checked

- Paper source:
  - `arxiv.tex:243` defines `r_theta(x,y) = beta log(pi_theta/pi_ref)`.
  - `arxiv.tex:422-445` defines `f-GRPO` through `psi(r_theta, a)` and `f-HAL = lambda FDO + (1-lambda) f-GRPO`.
  - `tables/fhal_alg.tex` gives the minibatch update with on-policy term `a_i (1+beta^{-1}) nabla psi(...)` and hybrid update `(1-lambda) g_on + lambda g_off`.
- Public repo:
  - cloned `https://github.com/rhaldarpurdue/f-GRPO` at HEAD `2102a871ce6e5e2ed815f210fac8fb1a036908ef`.
  - `src/UnslothFGRPO.py:496-499` computes `s_tokens = beta*(logp_new-logp_ref)` and `s_tokens2 = gamma*(logp_new-logp_old)`, then sets `s = s_tokens.sum(-1) + s_tokens2.sum(-1)`.
  - `src/train_fgrpo.py:476` exposes `--gamma` with default `1.0`.
  - `scripts/submit_single_fgrpo.sh:64` and `scripts/submit_single_fgrpo_safety.sh:64` explicitly pass `--gamma 1.0`.

## Why this matters

This is more than a cosmetic implementation detail. In the released trainer, all divergence-specific transforms (`kl`, `reverse_kl`, `pearson`, `hellinger`, `jensen_shannon`, `total_variation`) act on the augmented scalar `s`, not on the manuscript’s stated `r_theta`-only statistic. So the empirical pipeline the repo exposes is optimizing a materially different objective than the theorem-labeled one in the paper. That weakens the claim that the released artifact directly validates the divergence-estimation and reward-improvement interpretation argued in the manuscript.

## Decision consequence

I would score this as a meaningful reproducibility/soundness downgrade, not an automatic reject on its own. The core divergence-based idea may still be good, but the public artifact currently prevents a clean audit of whether the reported gains come from the stated `f-GRPO` / `f-HAL` objectives or from the augmented `+ gamma(log pi_theta-log pi_old)` implementation path.
