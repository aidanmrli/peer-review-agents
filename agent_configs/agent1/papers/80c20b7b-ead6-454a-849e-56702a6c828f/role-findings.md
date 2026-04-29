# MieDB-100k Reproducibility Findings

## Central claim and reproduction target

Targeted claim: the benchmark side of MieDB-100k is clinically curated and includes a human preference ranking (`Pref-Rank`) that helps validate the automated GPT-5.2 rubric.

## Paper and artifact evidence checked

- Paper source lines in `miedb_main.tex`:
  - lines 364-368: the paper says three people with clinical background manually curate 3,485 benchmark samples and check 6,000 train triplets with >95% viewed as high quality.
  - lines 430-437: the paper says GPT-5.2 provides `Rubric-S`, and three clinical evaluators also rank model outputs to produce `Pref-Rank`.
- Public artifact:
  - cloned `https://github.com/Raiiyf/MieDB-100k`
  - inspected `README.md`, `dataset_download.py`, `evaluation/`, `inference/`, and `OmniGen2-MIE/`

## Smallest meaningful reproducibility check actually run

- Verified the public repo contains:
  - dataset download helper for benchmark/train tar files
  - automatic evaluation scripts for DICE / PSNR / SSIM / VLM rubric scoring
  - inference scripts for several models
- Did **not** find released files for:
  - clinician ranking annotations or per-example `Pref-Rank` records
  - benchmark curation logs showing which 3,485 raw candidates were retained
  - agreement/correlation outputs linking human preference to the automated rubric

## Implementation or correctness risks

- The load-bearing benchmark claim is partly human-evaluation based, but the released artifact exposes only the automated evaluation path.
- Without the human ranking records, external reviewers cannot reproduce Table 3's `Pref-Rank` column or verify the implied validity link between `Rubric-S` and clinician preference.
- This is narrower than “the dataset is unusable”: image triplets and automatic scripts may still support model training and the automated benchmark, but the human-eval-backed benchmark claims remain only partially auditable.

## Novelty or framing context

- For dataset papers, benchmark value depends not only on scale but also on reproducible label provenance and evaluation transparency.
- Here the missing piece is not model code but benchmark supervision traceability.

## Decision impact

- Supports: the paper has a real dataset release and executable automatic evaluation path.
- Cannot verify: the clinician-curated and human-ranked portion of the benchmark.
- Score impact: moderate confidence reduction on benchmark reliability and on the claim that the automated rubric reflects clinically meaningful human preference.
