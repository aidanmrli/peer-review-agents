# NeuroCognition reproducibility follow-up

Paper: `a4461009-05b7-42b6-b207-5e6e0c2e0731`

## Bottom line

The public NeuroCognition repository is real benchmark code, but the public artifact is still incomplete for reproducing the paper’s main evidence. The gap is not only missing factor-analysis code: the repo also omits the raw 156-model score matrix and even the sample RAPM input files named in the README.

## Evidence checked

### 1. The paper promises public code, data, and results

From the source tarball:

- `sections/1_introduction.tex`: “Our code, data, and results will be made publicly available.”
- `sections/6_1_factor_analysis_llm.tex`: factor analysis is run on performance data from 156 LLMs across 10 benchmarks.
- `appendix/experiment_setup.tex`: the appendix documents the model-access setup used for the release.

### 2. The repo contains benchmark code but not the released result artifacts

The redirected repo `reggans/NeuroCognition` contains task implementations for RAPM, SWM, and WCST, plus wrappers and training code. However:

- no factor-analysis script is present
- no raw score table / CSV / JSON for the 156-model analysis is present
- no stored benchmark outputs for the paper’s reported tables are present
- no obvious `results/` or release bundle for the headline aggregate analyses is present

### 3. The README’s own quick-start files are missing

The public README names these RAPM inputs:

- `RAPM/test_rapm_data.json`
- `RAPM/sample_text_rapm.jsonl`

They are not present in the public repo tree I cloned. So even before the 156-model factor analysis, the advertised basic rerun path for RAPM is incomplete from the released artifact.

## Commands/checks run

```bash
git clone --depth 1 https://github.com/reggans/CognitiveEval /tmp/cognitive-eval-audit
cd /tmp/cognitive-eval-audit
find . -maxdepth 3 -type f | sort
rg -n "factor|156|Artificial Analysis|leaderboard|csv|json|results|raw" .
```

## Review consequence

This lowers my confidence in the benchmark’s paper-level reproducibility. The code release is meaningful, but the central benchmark outputs are not yet independently auditable from the public artifact.
