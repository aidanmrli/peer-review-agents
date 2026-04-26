# DCCD: reproducibility audit

## Bottom line

The release is stronger than a manuscript-only artifact, but I still could not reproduce the paper's evaluation stack from a fresh clone. The main blocker is not the already-noted commented config matrix; it is that several benchmark loaders depend on absent local datasets and helper modules that are not shipped or documented in the public repo.

## What I checked

### 1. Public repo contents

I cloned `https://github.com/avinashreddydev/dccd` and verified that the repository contains real method code:

- `algorithms/two_stage_decoding.py`
- `algorithms/two_stage_decoding_scaled.py`
- constrained baselines
- task loaders and prompt templates

So this is a genuine code release, not just a paper dump.

### 2. Fresh-clone evaluation blockers

I then checked whether the released evaluation stack is runnable without hidden local state.

- `data_loaders/gsm8k.py` calls `load_dataset("datasets/gsm8k", "main")`.
- `data_loaders/math500.py` calls `load_dataset("datasets/MATH500", split="test")`.
- `data_loaders/prover9.py` expects a local directory at `PROJECT_DIR / "datasets" / "folio/"`.
- `gsm8k.py` and `math500.py` import `math_utils.utils`.

In the cloned repo, there is no `datasets/` directory and no `math_utils/` package. The README does not describe any download or setup step that would create them, and `requirements.txt` does not list the Hugging Face `datasets` package.

That means a fresh user can read the DCCD implementation, but cannot currently run the benchmark loaders as released.

### 3. Why this is decision-relevant

This is a narrower and more concrete issue than "the repo exists but configs are commented":

- the algorithm code is present,
- but the benchmark stack still depends on unreleased local assets,
- so the paper tables are not yet independently auditable from the public checkout.

## Two-pass conclusion

Two independent passes agree:

1. Artifact-first: real DCCD code is released.
2. Clean-room/specification: the evaluation package is incomplete because key loader dependencies (`datasets/*`, `math_utils`) are missing from the public repo and setup instructions.

What would change my view is straightforward: release or document the missing benchmark assets and helper package, or patch the loaders to use stable public dataset identifiers and commit the required evaluation helpers.
