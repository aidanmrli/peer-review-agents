# MieDB-100k reply evidence

Paper: `80c20b7b-ead6-454a-849e-56702a6c828f`

## Bottom line

The public repo materially supports that MieDB-100k is a real release, but the paper's strongest quality-control phrasing appears broader than the source text justifies: the manuscript documents manual curation of the benchmark split and sampled QA on 6,000 training triplets, not exhaustive manual validation of all 100k+ triplets.

## Evidence checked

### Source text

- `miedb_main.tex` line 364: three people with clinical background manually evaluate and curate 3,485 representative benchmark samples from the raw test split.
- `miedb_main.tex` line 368: 6,000 training triplets are randomly selected for clinician evaluation, with over 95% viewed as high quality.
- `App/appendix.tex` line 74: benchmark split is manually curated; remaining training data is validated through sampling-based quality checks.
- `miedb_main.tex` line 528: conclusion claims "rigorous manual quality control to ensure clinical fidelity across all data."

### Public artifact

- Repo includes dataset download, OmniGen2-MIE training code, baseline inference scripts, and evaluation scripts, so this is not a placeholder release.
- I did not find public QA manifests, inter-rater statistics, or per-modality sampled error reports that would let an external reviewer verify broader manual-coverage claims.

## What I can verify

- The release is substantive.
- The paper supports benchmark manual curation and sampled training QA.
- The paper does not support a literal reading of manual clinical-fidelity assurance across the entire dataset.

## What remains unverified

- Exact annotator agreement.
- Stratified QA failure rates by modality/task.
- Whether any additional unpublished manual review covered the rest of the training set.

## Public reply content basis

I will post a narrow reply stating that the source and artifact together support a real release plus sampled QA, but that the manuscript should soften "across all data" unless broader audit records are available. This is decision-relevant because it changes the calibration of the paper's strongest fidelity claim without overstating the artifact weaknesses.
