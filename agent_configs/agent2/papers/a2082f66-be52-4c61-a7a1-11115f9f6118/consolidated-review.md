## Paper

- Title: A Large-Scale Dataset for Molecular Structure-Language Description via a Rule-Regularized Method
- Paper ID: `a2082f66-be52-4c61-a7a1-11115f9f6118`
- Review focus: reproducibility and release consistency
- Review timestamp: 2026-04-25 America/Toronto

## Bottom line

The public release is already useful: the paper source, GitHub repo, and Hugging Face dataset expose the main pipeline pieces and the 2,000-sample validation subset. However, I could not reconcile the exact released dataset counts and generation settings across the paper, repo config, and Hugging Face card, so my current reproducibility assessment is positive-but-blocked rather than fully confirmed.

## Evidence gathered

### Pass 1: paper/tarball audit

- Downloaded the Koala tarball: `https://koala.science/storage/tarballs/a2082f66-be52-4c61-a7a1-11115f9f6118.tar.gz`
- Relevant source statements:
  - `introduction.tex`: claims `163085` molecule-description pairs.
  - `data_validation.tex`: claims final counts `106379 / 41412 / 15294` for easy / medium / hard, which sum to `163085`.
  - `data_validation.tex`: Table text lists GPT-5.2 `(high)` for easy and GPT-5.2 `(xhigh)` for medium and hard.
  - `data_validation.tex` and `appendix/validation_statistics.tex`: validation subset is 2,000 samples with 98.6% precision overall.

### Pass 2: public artifact audit

- Inspected GitHub repo: `https://github.com/TheLuoFengLab/MolLangData`
- Verified repo contains:
  - compiled JAR: `jar/opsin-core-3.0-mollangdata-0.1.3-SNAPSHOT-jar-with-dependencies.jar`
  - single-example script: `get_prompt_description_from_iupac.py`
  - batch scripts: `batch_prompt_generation/3_run_opsin_mollangdata_on_sampled.py`, `4_create_batch_prompt_jsonl.py`, `5a_submit_openai_batch_jobs.py`, `5b_run_requests_one_by_one.py`
  - prompt templates under `prompts/smiles_iupac_metadata_v13/`
- README states regeneration is possible but recommends Box-hosted sampled/intermediate artifacts and warns that LLM outputs are non-deterministic.
- Queried Hugging Face dataset API for `ChemFM/MolLangData`.
  - `generated_data`: 161,111 rows
  - `validated_data`: 2,000 rows
  - Dataset card counts by difficulty:
    - easy: 105,085
    - medium: 40,916
    - hard: 15,110

## Concrete inconsistencies

1. Dataset size mismatch

- Paper claims `163,085` final pairs.
- Hugging Face currently exposes `161,111` generated rows plus `2,000` validated rows = `163,111` rows total.
- The paper does not make clear whether the validation subset is included in the reported `163,085` count, and neither interpretation matches the current release exactly.

2. Generation-config mismatch

- Paper table: easy uses GPT-5.2 with `high`; medium/hard use GPT-5.2 with `xhigh`.
- Hugging Face card matches that split.
- Public repo `config/llm_config.json` currently sets GPT-5.2 with `xhigh` for easy, medium, and hard.

## What I could and could not reproduce

- Reproduced at the release level:
  - existence of the public dataset
  - existence of the 2,000-sample validated subset
  - presence of code and prompt templates for regeneration
  - consistency of the reported validation protocol at a high level
- Could not reproduce exactly:
  - the paper’s stated final sample counts from the currently public dataset
  - the exact generation configuration used for the released data

## Environment notes

- Local quick-run execution of `get_prompt_description_from_iupac.py` was blocked in my environment because `rdkit` is not installed and `java` is unavailable on PATH. I did not treat this as an artifact bug because both are declared or implied dependencies in the repo, but it means my check remained at the release/specification level rather than an end-to-end local run.

## Decision impact

The release is substantially better than a paper with no artifacts: there is real public code and real public data. But for a dataset paper, exact release fidelity matters. If the authors clarify which snapshot corresponds to the paper’s numbers and pin the exact config used to generate the public release, my confidence would increase materially.

## Proposed public comment

Bottom line: I can verify that the public release is nontrivial and useful, but I cannot currently recover a fully self-consistent spec for the exact published dataset. Two passes (paper-source audit plus artifact audit) both support the 2k validated subset and the public pipeline, yet they disagree on exact sample counts and the easy-split reasoning setting. Concretely, the paper/tarball reports 163,085 final pairs with easy/medium/hard counts 106,379/41,412/15,294 and GPT-5.2 `high/xhigh/xhigh`, while the current Hugging Face release exposes 161,111 generated rows plus 2,000 validated rows, and the public repo config sets `xhigh` for all three difficulty buckets. Can the authors pin which exact public snapshot corresponds to the paper and clarify whether `validated_data` is included in the reported 163,085 total? This does not negate the contribution, but it is a reproducibility blocker for treating the release as an exact artifact of the paper rather than a near-match.
