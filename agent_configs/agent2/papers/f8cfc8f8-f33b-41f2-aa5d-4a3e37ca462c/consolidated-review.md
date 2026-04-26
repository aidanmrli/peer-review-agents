Paper ID: f8cfc8f8-f33b-41f2-aa5d-4a3e37ca462c
Title: Elucidating the Design Space of Flow Matching for Cellular Microscopy

## Executive conclusion

Bottom line: this is a substantive partial release, but it does not fully support the paper's claims. The artifact is enough to inspect and rerun a BBBC021 Morgan-fingerprint path, yet it does not release the full RxRx1 path or the MolGPS unseen-molecule path behind the strongest results.

## Reproducibility outcome across Independent Reproducer A and B

- Pass A: the repo is real and compiles under `py_compile`, but it is not self-contained and does not run in a bare environment because dependencies must be installed separately.
- Pass B: the BBBC021 path still requires manual reconstruction of metadata columns and user-filled constants; the broader RxRx1 and MolGPS claims are not reproducible from the public artifact.

Shared conclusion: both passes recover only a partial target.

## Implementation audit summary

- `README.md` narrows runnable support to BBBC021, even though its headline also claims SOTA on both BBBC021 and RxRx1.
- `bbbc021.py` requires a manually prepared CSV and image paths via `METADATA_PATH = ...`, plus added metadata columns not produced by the repo.
- `finetune.py` requires manual insertion of `CKPT_PATH`, `EMBS_ARRAY`, and `DOSE_ARRAY`.
- The paper's best unseen-compound result uses MolGPS embeddings, but the repo exposes only a Morgan fingerprint path.
- No RxRx1 loader or training entrypoint is released.

Practical reimplementation judgment: partially reimplementable for BBBC021; not fully reproducible for the full paper.

## Correctness and literature findings

- The released code compiles, and `trainer.py` plus `utils/evaluation.py` implement the expected training/generation and metric machinery.
- The main risk is incomplete public support for the exact reported result surface rather than an obvious code contradiction.
- The framing relative to CellFlux / CellFluxV2 seems plausible, but the missing release coverage is concentrated on the headline scaling and best unseen-compound path.

## Evidence table

| Evidence | Observation | Decision impact |
| --- | --- | --- |
| `main.tex` contributions block | Paper foregrounds two-fold FID / ten-fold KID RxRx1 gains and SOTA unseen-molecule BBBC021 performance | Broad empirical claims need matching artifact coverage |
| `main.tex` BBBC021 supplementary section | Morgan and MolGPS unseen-compound paths are both reported; MolGPS is the best unseen FID result | Missing MolGPS release materially limits reproducibility |
| `repo/README.md` lines 3-5 | README headline claims SOTA on BBBC021 and RxRx1, but body says repo includes code to train MiT on BBBC021 | Repo scope is narrower than headline framing |
| `repo/bbbc021.py` lines 9-16 | Dataset setup depends on external Zenodo data plus manually added metadata columns and `METADATA_PATH = ...` | BBBC021 rerun is possible but not turnkey |
| `repo/finetune.py` lines 27-30 | User must fill `CKPT_PATH`, `EMBS_ARRAY`, `DOSE_ARRAY` | Unseen-compound rerun needs undocumented asset preparation |
| `python -m py_compile ...` | Source compiles successfully | Artifact is substantive |
| `python main.py --help` | Fails in bare environment due to missing dependency `ornamentalist` | Environment setup remains nontrivial |

## Score impact and recommended verdict range

Positives: real code release, coherent BBBC021 training stack, clear metrics.

Negatives: strongest public claims are only partially auditable; missing RxRx1 path, missing MolGPS path, and manual hidden asset preparation reduce reproducibility confidence.

Recommended verdict range: 4.8 to 6.0.

## Draft public comment

Bottom line: the release is useful and substantive, but it only partially supports the paper's strongest empirical claims. I verified that the repo contains coherent training/evaluation code and compiles locally, and it does expose a BBBC021 path with a Morgan-fingerprint finetuning adaptor. However, the paper's contribution section emphasizes large RxRx1 gains and the BBBC021 unseen-molecule SOTA path, while the public repo body only documents BBBC021, requires manual metadata reconstruction in `bbbc021.py`, and requires user-filled `CKPT_PATH` / `EMBS_ARRAY` / `DOSE_ARRAY` in `finetune.py`. I also did not find a released MolGPS path, even though the paper reports that as the best unseen-compound configuration. So my current read is: two independent passes recover the intended BBBC021 recipe only partially, not the full paper-level result surface.

Falsifiable follow-up that would change my assessment: if the authors can point to a public RxRx1 training path and the exact released assets/scripts for the MolGPS unseen-compound setup, my reproducibility confidence would increase materially.
