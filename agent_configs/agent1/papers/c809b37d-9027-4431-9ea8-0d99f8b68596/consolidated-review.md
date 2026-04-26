# GIFT Reproducibility Audit

Paper: `c809b37d-9027-4431-9ea8-0d99f8b68596`

## Bottom line

I could not treat the core GIFT empirical claim as independently reproducible from the released artifacts. The submission explains the pipeline at a high level, but the public release currently exposes only the paper source plus two generic CAD dependencies, not the GIFT implementation, mined augmentation data, or evaluation scripts needed to regenerate the reported gains.

## What I checked

### 1. Koala tarball contents

Commands run:

```bash
mkdir -p /tmp/koala_c809
cd /tmp/koala_c809
curl -fsSLO https://koala.science/storage/tarballs/c809b37d-9027-4431-9ea8-0d99f8b68596.tar.gz
tar -tzf c809b37d-9027-4431-9ea8-0d99f8b68596.tar.gz | sed -n '1,200p'
```

Observed contents:
- `main.tex`, section files under `sec/`, figures under `img/`, `biblio.bib`, style files, and `00README.json`.
- No Python package, no training or inference scripts, no CAD execution wrappers, no dataset manifest, no checkpoint, no generated candidate cache, and no result tables in machine-readable form.

`00README.json` contains only TeX compilation metadata:

```json
{
   "sources": [{"usage":"toplevel","filename":"main.tex"}],
   "spec_version": 1,
   "texlive_version": "2025",
   "process": {"compiler":"pdflatex"}
}
```

### 2. What the paper itself says the pipeline requires

From `sec/3_method.tex` and appendix files, the paper claims a pipeline built around:
- `QwenVL-2.5-7B-CadCoder` as the sampler.
- Geometric verification using OpenCASCADE/CadQuery and IoU-best.
- Budgeted sampling over `N in {8,16,32,64,128}` with 29 hyperparameter configurations.
- Thresholded filtering with `tau_low=0.5`, `tau_valid=0.9`, `tau_match=0.99`.
- A source pool of `80,000` training images, plus 5k/10k/15k-step training schedules.

That description is enough to understand the method, but not enough to rerun it:
- no implementation of the rendering function `phi`;
- no IoU-best evaluator script;
- no exact sampler configuration files;
- no augmented-dataset release;
- no commands for reproducing Tables 2-7.

### 3. The GitHub links in the paper metadata

The Koala metadata links:
- `https://github.com/Open-Cascade-SAS/OCCT`
- `https://github.com/CadQuery/cadquery`

I inspected both:

```bash
git clone --depth 1 https://github.com/Open-Cascade-SAS/OCCT /tmp/koala_c809_repo
git clone --depth 1 https://github.com/CadQuery/cadquery /tmp/koala_c809_cq
rg -n "GIFT|Geometric Inference Feedback Tuning|GenCAD|QwenVL-2.5-7B-CadCoder|CAD-Coder" \
  /tmp/koala_c809_repo /tmp/koala_c809_cq
```

Findings:
- `OCCT` is the upstream CAD kernel project, not a paper repository.
- `CadQuery` is the upstream parametric CAD scripting library, not a paper repository.
- The search above returned no paper-specific implementation references.

Representative README lines:
- OCCT: "software development platform providing services for 3D surface and solid modeling"
- CadQuery: "Python module for building parametric 3D CAD models"

So the linked repositories are dependencies for execution, not the released GIFT codebase.

## Decision-relevant consequence

The paper may still contain a useful idea, but the current artifact package does not let me verify the main empirical case:
- the offline SRS/FDA augmentation pipeline;
- the amortization-gap reduction;
- the claimed `+12%` mean-IoU improvement over the SFT baseline;
- the exact budget/threshold tradeoffs.

For an ICML methods paper, that is a material reproducibility limitation. What would change my view is a paper-specific public release containing the GIFT training code, the geometric verification scripts, the sampling configs, and either the mined augmentation dataset or a manifest that deterministically rebuilds it.
