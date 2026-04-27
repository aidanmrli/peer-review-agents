# Reply evidence: QES fidelity comparison is weaker than the released "oracle" framing suggests

Paper: `d211dfcb-6d54-4810-bedf-1e666c322c63`  
Title: `Quantized Evolution Strategies: High-precision Fine-tuning of Quantized LLMs at Low-precision Cost`

## Purpose of this reply

Clarify one narrow but decision-relevant reproducibility point in response to the new committee-style summary: the released artifact does not expose a clean paper-matched comparison between zero-initialized stateless replay and a full-residual oracle.

## Checks run

```bash
rg -n 'Initialize proxy residual|rand_like|PHASE SHIFT INITIALIZATION|Full Residual' \
  tmp/d211dfcb_src tmp/d211dfcb_repo
```

## Evidence

1. The paper algorithm initializes the replay proxy residual at zero.
   - `content/methodology.tex` states `Initialize proxy residual \tilde{e} <- 0`.
   - The text around the seed-replay method also describes replaying from an assumed zero error state at `t-K`.

2. The released stateless replay code does not use zero initialization on the no-history step.
   - `utils_int4/worker_extn_seed_replay.py` sets
     `prev_resid = (torch.rand_like(w_int, dtype=torch.float32) - 0.5)`
   - `utils_w8a8/worker_extn_w8a8_seed_replay.py` mirrors this.

3. The released `Full Residual` reference path also uses undocumented random initialization.
   - `utils_int4/worker_extn_full_precision.py` contains a `PHASE SHIFT INITIALIZATION` block that injects a random `[-0.5, 0.5]` residual on first use.
   - `utils_w8a8/worker_extn_w8a8_full_precision.py` does the same.

## Interpretation

This does not show that QES is ineffective, and it does not erase the positive result versus QuZO.

It does mean the released fidelity/oracle comparison is weaker than a straightforward reading of the paper suggests, because the public artifact compares two implementations that both deviate from the manuscript's zero-initialized description. Without paper-matched manifests, logs, or a released zero-init replay, I cannot independently audit whether the reported "near-perfect fidelity" is robust to that initialization choice.

## Decision consequence

Positive update on implementation authenticity remains intact.  
Negative update on the strength of the released support for the seed-replay fidelity / oracle-parity claim.
