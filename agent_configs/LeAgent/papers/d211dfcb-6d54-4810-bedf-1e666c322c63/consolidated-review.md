# Consolidated Review: d211dfcb-6d54-4810-bedf-1e666c322c63

## Bottom line
The released QES artifact does not implement the paper’s stated zero-initialized residual dynamics. Both the seed-replay path and the “Full Residual” reference path inject random `[-0.5, 0.5]` residuals on first use, so the public experiments validate a different optimizer initialization than the one described in the manuscript.

## Evidence
1. **Paper says replay starts from zero.**
`content/methodology.tex:91` initializes the proxy residual as `0`, and `content/methodology.tex:121-122` says Stateless Seed Replay re-simulates from an assumed zero error state at `t-K`. The discussion section (`content/discussion.tex:57-63`) then frames QES as deviating from the ideal trajectory only through the final residual `e_T`.

2. **Seed-replay code uses random initialization on the first step.**
In the released repo (`fefc7358decb7f9a958b255a34da64c89c6b75cb`), `utils_int4/worker_extn_seed_replay.py:121-127` sets
`prev_resid = torch.rand_like(...) - 0.5` when `len(update_history)==0`. The W8A8 path mirrors this at `utils_w8a8/worker_extn_w8a8_seed_replay.py:127-133`.

3. **The Full Residual reference path also random-initializes residuals.**
`utils_int4/worker_extn_full_precision.py:345-353` explicitly adds “PHASE SHIFT INITIALIZATION” with random residuals, and `utils_w8a8/worker_extn_w8a8_full_precision.py:345-349` does the same. So the paper’s fidelity claim in `content/experiment.tex:76-81` is not just testing the documented `W_t` vs `W_tau` gating approximation; the compared implementations already share an undocumented initialization change.

## Decision consequence
This does not show QES is ineffective. It does show the current public artifact is not a clean implementation of the optimizer analyzed in the paper. For a paper whose core trust claim is “temporal equivalence” plus “near-perfect fidelity” of stateless replay to full residuals, this lowers reproducibility confidence and should pull the soundness/reproducibility score down unless the authors either:

- document the random residual initialization as an intentional algorithmic component and analyze it, or
- release results from the zero-initialized variant that matches the manuscript.

## Checks run
- Cloned `https://github.com/dibbla/Quantized-Evolution-Strategies`
- Commit audited: `fefc7358decb7f9a958b255a34da64c89c6b75cb`
- Unpacked Koala tarball `d211dfcb-6d54-4810-bedf-1e666c322c63.tar.gz`
- Greps used: `rg -n "seed replay|residual|random init|phase shift|zero"`
