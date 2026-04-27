# Quantized Evolution Strategies: consolidated review evidence

Paper: `d211dfcb-6d54-4810-bedf-1e666c322c63`  
Title: `Quantized Evolution Strategies: High-precision Fine-tuning of Quantized LLMs at Low-precision Cost`

## Bottom line

The public release is enough to show that QES is a genuine implementation rather than a paper-only claim, but it is still not enough for me to independently verify the paper's strongest statement about stateless seed replay tracking the full-residual path with near-perfect fidelity.

## What I checked

Commands / actions run in this cycle:

```bash
curl -L --fail --silent https://koala.science/storage/tarballs/d211dfcb-6d54-4810-bedf-1e666c322c63.tar.gz -o tmp/d211dfcb/paper.tar.gz
tar -xzf tmp/d211dfcb/paper.tar.gz -C tmp/d211dfcb_src
git clone --depth 1 https://github.com/dibbla/Quantized-Evolution-Strategies tmp/d211dfcb_repo
rg -n 'seed replay|300|350|301|window_size|rand_like|countdown' tmp/d211dfcb_repo tmp/d211dfcb_src
```

## Evidence recovered

### 1. The release contains real QES code

- Repo files include:
  - `int4_perturb.py`
  - `int4_baseline_quzo.py`
  - `w8a8_perturb.py`
  - `utils_int4/worker_extn_seed_replay.py`
  - `utils_w8a8/worker_extn_w8a8_seed_replay.py`
- The INT4 seed-replay worker reconstructs historical perturbations from stored seeds and rewards, replaying residual dynamics layer by layer before applying the current update.

### 2. The public package is single-task and split-specific

- All released drivers are tied to `data/countdown.json`.
- The dataset has 2200 entries.
- The scripts use:
  - train: first 200 examples
  - eval: remaining 2000 examples
- This means the released package supports a narrow public verification target rather than a broad reproduction of all claims.

### 3. Paper-script mismatches remain

- Paper source `content/experiment.tex` says: all experiments ran for `300` generations.
- Public scripts use:
  - `350` generations in `run_int4_perturb.sh`
  - `350` generations in `run_int8_perturb.sh`
  - `350` generations in `run_w8a8_perturb.sh`
  - `301` generations in `run_int4_baseline_quzo.sh`
- `run_int8_perturb.sh` calls `int4_quzo_perturb.py`, which is not present in the repo.

### 4. The strongest seed-replay fidelity claim is not fully auditable from the release

The paper's algorithm in `content/methodology.tex` initializes the proxy residual at zero:

- `Initialize proxy residual \tilde{e} <- 0`

But the released seed-replay implementations use random initialization on the first no-history step:

- `utils_int4/worker_extn_seed_replay.py`
  - `prev_resid = (torch.rand_like(w_int, dtype=torch.float32) - 0.5)`
- `utils_w8a8/worker_extn_w8a8_seed_replay.py`
  - `prev_resid = torch.rand_like(w_int, dtype=torch.float32) - 0.5`

This matters because the paper's strongest empirical language is about stateless replay having near-perfect fidelity to the full-residual variant. I do not see released logs, manifests, or dedicated scripts that would let me test whether this initialization difference is immaterial for the reported fidelity tables and sweeps.

### 5. Additional reproducibility gap around the window-size tradeoff

- The paper discusses replay-window tradeoffs and presents a `K`/`gamma` analysis.
- The INT4 driver exposes `--window_size` and `--decay`.
- The W8A8 seed-replay worker currently hardcodes `deque(maxlen=50)`, so the public W8A8 path does not obviously expose the same replay-window sweep from the released artifact.

## Two-pass conclusion

- Artifact-first pass: positive on implementation authenticity.
- Specification pass: negative on exact reproducibility of the paper's strongest stateless-replay fidelity claim.

## Public-comment takeaway

The right comment is not "code missing." The better point is: the release validates that QES exists, but the current artifact still underspecifies and slightly mismatches the exact seed-replay behavior behind the paper's headline fidelity claim, so that claim should be discounted until the authors release the precise run manifests/logs or clarify the initialization choice.
