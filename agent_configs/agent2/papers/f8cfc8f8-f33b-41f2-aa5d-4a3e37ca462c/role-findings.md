Paper ID: f8cfc8f8-f33b-41f2-aa5d-4a3e37ca462c
Title: Elucidating the Design Space of Flow Matching for Cellular Microscopy

## Reproducibility lead: central claim and reproduction target

Central claim checked: the paper presents a simple flow-matching recipe that delivers large RxRx1 gains and strong BBBC021 seen/unseen perturbation results, with public code.

Reproduction target: determine whether the release is sufficient to rerun the BBBC021 path and whether it substantively supports the broader RxRx1 and unseen-molecule claims.

Evidence examined:
- `main.tex`
- `README.md`, `main.py`, `finetune.py`, `bbbc021.py`, `trainer.py`, `utils/evaluation.py`
- `python -m py_compile main.py finetune.py bbbc021.py model.py trainer.py adapters.py cellflux.py utils/*.py`

Findings:
- The release is real code and compiles.
- The paper foregrounds two headline empirical claims: large RxRx1 gains and BBBC021 unseen-molecule SOTA.
- The repo is narrower than that scope: `README.md` says it includes code to train MiT on BBBC021, but no parallel RxRx1 path is released.
- BBBC021 reruns still require manual data reconstruction: `bbbc021.py` leaves `METADATA_PATH = ...` and assumes extra metadata columns already exist.

Decision impact:
- Positive: enough code exists to understand the intended BBBC021 pipeline.
- Negative: the artifact only partially supports the paper's headline reproducibility claims.

## Reproducer A: artifact-first check

Task scope: inspect what can be run from the released artifact.

Commands/checks:
- Downloaded Koala PDF and tarball.
- Cloned `https://github.com/valence-labs/microscopy-flow-matching`.
- Ran `python -m py_compile ...`.
- Attempted `python main.py --help`.

Findings:
- Static compilation succeeded.
- Direct execution failed in a bare environment because required packages are not bundled (`ModuleNotFoundError: ornamentalist`).
- The README documents only BBBC021 execution, with a 16-GPU claim for the seen-compound result.
- No checkpoints, logs, or pinned run artifacts are included.

Decision impact:
- Usable for inspection and likely runnable after setup, but not close to turnkey reproduction.

## Reproducer B: clean-room/specification check

Task scope: judge whether the paper plus release specify enough to reconstruct the reported numbers.

Findings:
- BBBC021 preprocessing depends on an external IMPA reproducibility drop plus user-added metadata columns.
- Unseen-compound Morgan finetuning requires user-filled `CKPT_PATH`, `EMBS_ARRAY`, and `DOSE_ARRAY`.
- The paper reports a stronger MolGPS unseen-compound result, but the repo only exposes a Morgan path.
- The paper attributes large RxRx1 improvements, but the repo contains no RxRx1 dataset loader or training entrypoint.

Decision impact:
- A clean-room reproducer can probably reconstruct the BBBC021 Morgan path with work.
- The full paper, especially RxRx1 and MolGPS claims, is not reproducible from the released artifact as-is.

## Implementation auditor: code/artifact/repo match

Findings:
- `trainer.py` contains generation and FID/KID-style evaluation code.
- `finetune.py` matches the described Morgan adaptor idea.
- The README headline claims SOTA on BBBC021 and RxRx1, but the body only promises code for BBBC021.
- The paper highlights MolGPS unseen-compound SOTA, but no MolGPS path is released.
- The artifact does not include metadata-generation scripts, checkpoints, or exact split manifests.

Decision impact:
- Implementation supports a subset of the manuscript, not the full result surface.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

Findings:
- `utils/evaluation.py` implements the expected metric family, which is a strength.
- Exact metric reproduction still depends on external downloaded models and the same preprocessing/manifests used by the authors.
- Because the repo omits the stronger MolGPS path and RxRx1 path, the paper's broad SOTA conclusion is only partially auditable from release.

Decision impact:
- No obvious code contradiction.
- Empirical conclusions are under-supported by the public artifact breadth.

## Literature specialist: novelty/framing against permitted prior work

Findings:
- The framing against CellFlux / CellFluxV2 and prior microscopy generation work is plausible.
- The distinctive contribution seems to be the ablation-and-scaling recipe plus unseen-compound adaptor study.
- Since the missing public coverage is concentrated on the headline scaling and best unseen-compound path, the practical novelty is harder to verify than the prose suggests.

Decision impact:
- Novelty is plausible, but reproducibility evidence does not fully substantiate the broad framing.

## Limitations or blockers

- `pdftotext` was unavailable; I inspected the LaTeX source instead.
- I did not install the full dependency stack or run multi-GPU training.
- No author logs/checkpoints were included.

## Confidence

Moderate-high on artifact coverage findings; moderate on ultimate paper quality.
