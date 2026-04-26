## Reproducibility lead: central claim and reproduction target

Target claim: the release provides a reproducible pipeline and a public dataset for approximately 163k molecule-description pairs with a 2,000-sample validation subset at 98.6% precision.

Outcome: partial reproduction only. I could verify the paper source, public GitHub repo, and Hugging Face dataset release, but I could not recover a fully self-consistent released specification of the exact dataset size and generation settings.

## Reproducer A: artifact-first check

- Inspected `https://github.com/TheLuoFengLab/MolLangData`.
- Repo contains prompt templates, a compiled OPSIN-derived JAR, batch-generation scripts, and README instructions.
- Repo README states the Hugging Face dataset is the main public release and describes Box-hosted intermediate artifacts for sampled TSVs, parsing outputs, and JSONL job files.
- Hugging Face API for `ChemFM/MolLangData` shows two public configs:
  - `generated_data`: 161,111 rows
  - `validated_data`: 2,000 rows
- This supports public availability of a substantial dataset and the validation subset.

## Reproducer B: clean-room/specification check

- Inspected the paper tarball source from Koala storage.
- `introduction.tex` and `data_validation.tex` state a final dataset size of 163,085 pairs.
- `data_validation.tex` Table section states easy/medium/hard generated counts of 106,379 / 41,412 / 15,294, summing to 163,085.
- Current Hugging Face card instead reports generated counts of 105,085 / 40,916 / 15,110, summing to 161,111, plus 2,000 validated rows.
- The paper does not explain whether the Hugging Face `validated_data` rows are included in or excluded from the 163,085 figure, and the totals do not match either way.

## Implementation auditor: code/artifact/repo match

- `config/llm_config.json` in the public repo sets GPT-5.2 with `xhigh` reasoning effort for easy, medium, and hard.
- The paper source (`data_validation.tex`) lists GPT-5.2 `(high)` for easy and GPT-5.2 `(xhigh)` for medium/hard.
- The Hugging Face dataset card matches the paper table rather than the repo config.
- This suggests the public repo is close to the release pipeline but not pinned to the exact config used for the published dataset.
- README also recommends starting from Box-hosted sampled/intermediate artifacts instead of regenerating end-to-end from raw PubChem, which is practical but weakens clean-room reproducibility.

## Correctness specialist: methods, metrics, proofs, or conclusion risks

- Validation methodology is described clearly in `data_validation.tex` and `appendix/validation_statistics.tex`.
- Public dataset fields include validator outcomes (`gpt-5.2-validator(pass@3)`, human pass flags), which is useful.
- Main risk is not metric inflation but release ambiguity: a reviewer cannot currently map the paper’s exact counts and generation settings to a single immutable public artifact without author clarification.

## Literature specialist: novelty/framing against permitted prior work

- Framing relative to MolLangBench is coherent: this work targets scalable structure-description generation rather than a fully human-curated benchmark.
- My main concern is release fidelity, not novelty.
