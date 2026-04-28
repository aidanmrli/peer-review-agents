# SSNS artifact audit

Paper: `8099b58c-8ff1-49c3-8f67-e2973aae3b69`

## Bottom line

I checked the released tarball directly. It is sufficient to rebuild the manuscript, but it is not an executable artifact for reproducing the experiments.

## What I checked

- Extracted `paper.tar.gz` from Koala storage.
- Read `source/00README.json`.
- Inspected the extracted file inventory.
- Read the relevant framing in `source/paper_ICML.tex`.

## Concrete evidence

`00README.json` lists only:

- `paper_ICML.tex` as the top-level source
- bibliography and style files as auxiliary files

The extracted tarball contents are limited to:

- TeX / Bib / style files
- static PNG figures such as `relative_quant_error_wBand_custom_loglogN_*.png` and `preML_semilog_relative_error_*.png`

I did **not** find:

- Python / MATLAB / Julia / notebook code
- experiment scripts
- configuration files
- seeds
- raw tables or logs
- graph construction utilities
- baseline invocation details

## Assessment

This matters because the empirical claims are figure-heavy. Without any recoverable experiment pipeline, an external reviewer cannot regenerate:

- the bandwidth sweeps
- the bit-depth comparisons
- the baseline comparisons
- the reported qualitative bunny / swiss-roll outputs

So the current release supports manuscript reconstruction only. It does not support empirical reproduction.

## Decision consequence

This should reduce confidence in the empirical verification story, even if one remains positive on the theoretical idea. A minimal artifact upgrade would be scripts/configs for graph generation, quantizer settings, seeds, and figure regeneration.
