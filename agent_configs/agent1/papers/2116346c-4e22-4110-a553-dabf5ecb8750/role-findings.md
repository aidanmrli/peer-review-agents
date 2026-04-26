## Reproducibility lead

Central claim and reproduction target: the release should let an independent reviewer rerun the benchmark sweeps behind the SynthSAEBench-16k results, especially the main L0 comparison across SAE families.

Bottom line: the public repo is functional but the main benchmark entrypoint is not runnable from a clean checkout because it assumes a local benchmark-model directory that is not included and is not fetched automatically.

## Reproducer A

Artifact-first check:

- Cloned `https://github.com/decoderesearch/synth-sae-bench-experiments`.
- Verified the checkout does **not** contain a top-level `synth-sae-bench-16k/` directory.
- Read `experiments/sweeps/sweep_l0.py`: it hardcodes `MODEL_PATH = ... / "synth-sae-bench-16k"` and passes that path into every runner config.
- Read `README.md`: it documents `SyntheticModel.from_pretrained("decoderesearch/synth-sae-bench-16k-v1")`, but it does not document a step that materializes that model into the local path expected by `sweep_l0.py`.

Result: a reviewer following the repo README can run the superposition sweep example, but does not get a documented path to run the paper's main L0 sweep from a clean clone.

## Reproducer B

Clean-room/specification check:

- The repo includes `experiments/create_synth_bench_16k_model.py`, which saves a local `synth-sae-bench-16k` directory.
- That script constructs the model with `device="cuda"` and no fallback or CLI options.
- The README does not point readers to this script as a prerequisite for `sweep_l0.py`.

Result: the only visible path to satisfy `sweep_l0.py`'s local-path assumption is an undocumented script with a hard CUDA requirement.

## Implementation auditor

Code/artifact/repo match:

- `README.md` says the benchmark model is on Hugging Face and shows `from_pretrained(...)`.
- `sweep_l0.py` does not use `from_pretrained`; it expects a pre-existing filesystem artifact.
- `sweep_superposition.py` is self-contained because it builds `SyntheticModelConfig` inline, so the reproducibility gap is specific to the paper's main benchmark sweep rather than the entire repo.

## Correctness specialist

Methods/metrics/conclusion risks:

- This is primarily a reproducibility risk, not a theorem/proof issue.
- Without a documented, clean-start path for `sweep_l0.py`, the central architecture-comparison result is harder to verify independently than the paper presentation suggests.

## Literature specialist

Novelty/framing against permitted prior work:

- Existing thread participants already covered novelty framing, metric bias, and downstream-validity concerns.
- The new contribution here is narrower: the release path for the benchmark artifact itself is incomplete for the main experiment.
