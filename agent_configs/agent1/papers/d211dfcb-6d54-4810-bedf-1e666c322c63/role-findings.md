# Quantized Evolution Strategies: role findings

## Reproducibility lead: central claim and reproduction target
- Central claim: QES enables direct fine-tuning in quantized weight space and the stateless seed-replay variant tracks the full-residual variant with near-perfect fidelity while using inference-level memory.
- Reproduction target for this cycle: verify whether the public artifact supports the strongest seed-replay fidelity claim in the paper, not just whether some QES code exists.

## Reproducer A: artifact-first check
- Cloned `https://github.com/dibbla/Quantized-Evolution-Strategies`.
- Repo contains runnable-looking drivers: `int4_perturb.py`, `int4_baseline_quzo.py`, `w8a8_perturb.py`, worker extensions under `utils_int4/` and `utils_w8a8/`, and `data/countdown.json`.
- `data/countdown.json` has 2200 items. The public scripts train on `[:200]` and evaluate on `[200:]` in `int4_perturb.py`, `int4_baseline_quzo.py`, and `w8a8_perturb.py`.
- One release path is broken as shipped: `run_int8_perturb.sh` calls `int4_quzo_perturb.py`, which is absent.

## Reproducer B: clean-room/specification check
- Paper source says all experiments ran for 300 generations in `content/experiment.tex`.
- Public run scripts use 350 generations for QES (`run_int4_perturb.sh`, `run_int8_perturb.sh`, `run_w8a8_perturb.sh`) and 301 for the QuZO baseline (`run_int4_baseline_quzo.sh`).
- Paper Algorithm `Stateless QES Update with Seed Replay` initializes proxy residual `\\tilde{e} <- 0`.
- Released implementations instead use random initialization on the no-history step:
  - `utils_int4/worker_extn_seed_replay.py`: `prev_resid = (torch.rand_like(w_int, dtype=torch.float32) - 0.5)`
  - `utils_w8a8/worker_extn_w8a8_seed_replay.py`: `prev_resid = torch.rand_like(w_int, dtype=torch.float32) - 0.5`

## Implementation auditor: code/artifact/repo match
- Positive: the seed-replay mechanism is substantive, not a placeholder. `utils_int4/worker_extn_seed_replay.py` reconstructs historical perturbations from seeds, replays residual dynamics, and applies boundary gating using current weights as the approximation described in the paper.
- Mismatch: the released code does not exactly match the paper’s algorithm at initialization, which is relevant because the paper’s strongest empirical claim is fidelity to the full-residual path.
- The public repo does not expose logs, manifests, or dedicated scripts for the paper’s fidelity analysis and window-size/decay analysis; I only found the source text and general training scripts.
- W8A8 seed replay hardcodes `deque(maxlen=50)` in the worker extension, so the released W8A8 path does not obviously expose the paper’s window-size tradeoff as a public knob.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
- The strongest paper language is about temporal equivalence / near-perfect fidelity of stateless replay to full residual.
- Because the artifact changes the first-step initialization from zero to random and does not release the specific evidence behind the fidelity table/sweep, I cannot independently verify that the observed parity is robust rather than configuration-sensitive.
- This is not a fatal implementation-authenticity issue; it is a support gap for the strongest claim.

## Literature specialist: novelty/framing against permitted prior work
- Framing against quantized ZO baselines and error-feedback ideas appears plausible from the paper text.
- I did not use post-publication signals or external reviews. No further novelty audit this cycle beyond checking whether the released code supports the paper’s specific seed-replay claims.
