# Independent Reproducer A Report

Paper: `d50ca57f-ac9a-438f-b0f5-fab02c8d64df`, "Transport Clustering: Solving Low-Rank Optimal Transport via Clustering"

Role: Independent Reproducer A

Date: 2026-04-24

## Scope and permitted sources

I used only the official paper artifacts in `papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts`, the assignment-provided Koala metadata, and prior-work references as cited inside the paper source. I did not consult OpenReview, citation counts, decisions, social media, or any later discussion. I did not read Independent Reproducer B's report before completing this pass.

The Koala metadata supplied for this task says the paper is `in_review`, has `artifacts/paper.pdf` and `artifacts/main.tex`, has no linked GitHub repository, and has only two existing comments: one bibliography-format check and one positive general review.

## Claim attempted

Primary empirical claim attempted: the paper claims Transport Clustering (TC) is empirically better than existing low-rank OT solvers on synthetic benchmarks and large-scale high-dimensional datasets. In the abstract and contribution paragraph, the authors state that TC "empirically obtains lower transport cost than existing low-rank OT solvers" and outperforms existing methods on synthetic and real datasets. In the experiment section, they claim:

- synthetic validation: TC is consistently best on low-rank OT cost across three synthetic benchmarks, with reported average relative improvements of 23 percent on shifted Gaussians and 4 percent on SBM over the next-best method (`main.tex` lines 1076-1112);
- CIFAR-10 and single-cell: TC has lower OT cost and better AMI/ARI/CTA than LOT and FRLC in the reported tables (`main.tex` lines 1114-1146, 2702 onward);
- W2 estimation: TC gives the most accurate low-rank estimate of the fragmented-hypercube Wasserstein distance for most sample sizes (`main.tex` lines 1148-1166 and 2600-2632).

Smallest feasible independent reproduction from official artifacts: since no executable code, result data, or repository is included, I attempted two checks:

1. Determine whether the official artifact bundle contains enough code, commands, data provenance, seeds, numerical arrays, or solver settings to reproduce the empirical claims.
2. Independently verify the analytic target value used in the fragmented-hypercube W2 benchmark, namely `W_2^2 = 8`, and check simple arithmetic consistency for selected reported tables.

## Setup used

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Environment:

```bash
date -Is && uname -a && python3 --version
```

Observed:

```text
2026-04-24T19:21:44-04:00
Linux cn-f004.server.mila.quebec 5.15.0-173-generic #183-Ubuntu SMP Fri Mar 6 13:29:34 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
Python 3.12.12
```

Available/relevant tools checked:

```bash
which rg sed awk strings gs qpdf mutool exiftool pdfimages python3 pdflatex latexmk
```

Observed relevant availability: `rg`, `sed`, `awk`, `strings`, `python3`, `pdflatex`, and `latexmk` were present. `pdftotext`, `pdfinfo`, `gs`, `qpdf`, `mutool`, `exiftool`, and `pdfimages` were not found. I therefore used the LaTeX source and string inspection rather than PDF text extraction. I did not compile the LaTeX source because the assigned task was to inspect reproducibility of claims, not regenerate the manuscript, and because the user instructed that only this report file should be written.

## Commands and observations

### Artifact inventory

```bash
find papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts -maxdepth 2 -type f -printf '%p %s bytes\n' | sort
tar -tzf papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/source.tar.gz | sort | sed -n '1,240p'
sed -n '1,220p' papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/00README.json
```

Observed: the source bundle contains `main.tex`, `references.bib`, ICML style files, auxiliary style files, `paper.pdf`, and pre-rendered figure PDFs. The tarball lists the same LaTeX/figure assets. I found no Python/JAX scripts, notebooks, raw generated data, CSV/NPY/JSON result arrays, solver configuration files, environment files, shell commands, or linked repository.

The `00README.json` only describes TeX compilation:

```json
{
  "sources": [{"usage": "toplevel", "filename": "main.tex"}, ...],
  "texlive_version": "2025",
  "process": {"compiler": "pdflatex"}
}
```

### Source search for reproducibility hooks

```bash
rg -n "(code|github|repo|repository|data|dataset|seed|random|hyper|parameter|Table|Figure|fig:|Algorithm|runtime|experiment|implementation|Appendix|theorem|proposition|proof|complexity|synthetic|cluster|low-rank|rank|N=|epsilon|regulari|iteration|convergence|baseline|POT|OTT|scikit|sklearn|python|command|script)" \
  papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/main.tex \
  papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/references.bib
```

Key reproducibility-relevant passages:

- Algorithm 1, Transport Clustering, is specified mathematically but not as runnable code (`main.tex` lines 593-608).
- Implementation details state synthetic Monge maps use `ott-jax` Sinkhorn with `epsilon=10^{-5}` and 10,000 maximum iterations, real data uses `HiRef`, GKMS uses JAX, step size `gamma_k = 2`, 250 iterations, scikit-learn K-means initialization, and a 50/50 random centering mixture (`main.tex` lines 2478-2508).
- Synthetic data recipes are described in prose (`main.tex` lines 2513-2580), including several random draws and five seeds `s in {1,2,3,4,5}` (`main.tex` lines 2520-2524).
- CIFAR-10 specifies ResNet `resnet18-f37072fd.pth`, PCA to 50 dimensions, a stratified 50/50 split, and a fixed seed, but does not give the fixed seed value or preprocessing code (`main.tex` lines 2582-2586).
- Single-cell preprocessing describes `scanpy` steps, h5ad files, `df_cell.csv`, randomized PCA, and subsampling to divisibility-friendly sizes, but no exact download command, file version/hash, subsampling rule, or seed is present (`main.tex` lines 2596-2598).
- Table values are embedded in the paper source, but I found no raw trial-level outputs behind the plotted figures or averaged numbers (`main.tex` lines 2612-2700 and 2702 onward).

### Figure/result-data inspection

```bash
for f in papers/d50ca57f-ac9a-438f-b0f5-fab02c8d64df/artifacts/figures/*.pdf; do
  printf '%s\n' "$f"
  strings "$f" | rg -n "(TC|LOT|FRLC|LIN|Cost|Rank|AMI|ARI|epsilon|varepsilon|CIFAR|Transport|Clustering|[0-9]+\.[0-9]+)" | sed -n '1,30p'
done
```

Observed: several figures were produced by Matplotlib, e.g. `epsilon_sensitivity.pdf` reports `Matplotlib v3.10.3` and several synthetic figures report `Matplotlib v3.8.4`, but the PDFs do not expose clean source arrays or enough structured data to recover the reported curves. The figure PDFs are therefore presentation artifacts, not reproducible result artifacts.

### Analytic W2 benchmark check

Paper source claim (`main.tex` lines 2602-2610): for the fragmented hypercube benchmark, `P_0 = Unif([-1,1]^d)`, `P_1 = T_# P_0`, and

```text
T(X) = X + 2 * sgn(X) o (e_1 + e_2)
```

The paper states that Brenier's theorem gives target value `W_2^2 = 8`.

Manual derivation: for every sample `X` away from the measure-zero coordinate hyperplanes, only the first two coordinates move. The displacement is

```text
T(X) - X = 2 * sgn(X) o (e_1 + e_2),
```

so

```text
||T(X) - X||_2^2 = 2^2 + 2^2 = 8.
```

The map is monotone coordinatewise and can be viewed as a gradient of the convex potential `0.5 ||x||_2^2 + 2|x_1| + 2|x_2|`, matching the paper's Brenier-theorem justification for squared Euclidean OT. Thus the target value is analytically reproducible from the paper text.

Lightweight numerical sanity check without NumPy:

```bash
python3 - <<'PY'
import random
random.seed(0)
vals=[]
for _ in range(10000):
    x0=random.uniform(-1,1)
    x1=random.uniform(-1,1)
    d0=2*(1 if x0>=0 else -1)
    d1=2*(1 if x1>=0 else -1)
    vals.append(d0*d0+d1*d1)
print('fragmented_hypercube_displacement_squared_mean', sum(vals)/len(vals))
print('unique_displacement_squared', sorted(set(vals)))
print('epsilon_1e0_over_1e-5', 9.576/5.050)
print('epsilon_1e1_over_1e-5', 14.538/5.050)
print('TC_init_FRLC_rand_over_TC_at_r250', 8.4448/7.0762)
print('Kant_Y64_FRLC_minus_TC', 10.508-8.983, 'LOT_minus_TC', 10.108-8.983)
PY
```

Observed:

```text
fragmented_hypercube_displacement_squared_mean 8.0
unique_displacement_squared [8]
epsilon_1e0_over_1e-5 1.8962376237623764
epsilon_1e1_over_1e-5 2.878811881188119
TC_init_FRLC_rand_over_TC_at_r250 1.193408891778073
Kant_Y64_FRLC_minus_TC 1.5249999999999986 LOT_minus_TC 1.125
```

This matches the analytic target `W_2^2 = 8`. The arithmetic behind one ablation statement is also directionally consistent: the table's `epsilon=10^0` cost is about 1.90 times the `epsilon=10^-5` cost, and `epsilon=10^1` is about 2.88 times larger, close to the paper's prose "factor of two" and "factor of three" statement (`main.tex` lines 2658-2678). This is not a reproduction of the experiment, only a consistency check of reported table values.

## Reproduction outcome

Outcome for the central empirical claim: blocked.

The official artifacts do not allow me to independently rerun TC, LOT, FRLC, LIN, HiRef, GKMS, the CIFAR pipeline, or the single-cell pipeline. The paper provides high-level algorithmic equations and some hyperparameters, but not the implementation or enough exact procedural state to reproduce the numbers.

Outcome for the smallest analytic claim attempted: match.

The fragmented-hypercube target value `W_2^2 = 8` is independently derivable from the provided formula and verified by a small deterministic displacement calculation.

Outcome for table/figure consistency: partial match.

The reported tables in `main.tex` are internally consistent for the specific arithmetic checks above, but these checks only verify arithmetic on reported numbers. They do not verify that the numbers came from the described experiments.

## Concrete blockers

- No linked GitHub repository is provided in the Koala metadata, and no code repository URL appears in the artifact bundle.
- `source.tar.gz` contains LaTeX and pre-rendered figure PDFs only. It does not contain experiment scripts, notebook workflows, result tables in machine-readable form, figure-generation code, environment files, or raw outputs.
- The central method implementation is described as a JAX implementation of `GKMS`, but the implementation is not included.
- The full-rank registration step depends on `ott-jax` and `HiRef`; no exact versions, commands, invocation scripts, tolerance criteria, or solver logs are provided.
- Synthetic experiments specify seeds `1..5`, but not all generator settings are precise enough to rerun. For example, random partition sampling for shifted Gaussians is underspecified, and exact library versions/defaults are absent.
- CIFAR-10 reproduction is blocked by the missing fixed seed value, missing feature-extraction/PCA code, missing preprocessing details, and missing solver invocation.
- Single-cell reproduction is blocked by absent download/version identifiers for the h5ad files and metadata, missing exact subsampling procedure, missing randomized PCA seed, missing balanced-class sampling rule, and missing solver commands.
- Figure PDFs are generated presentation artifacts. They do not expose raw data arrays sufficient to recover the plotted curves or reported averages.

## Confidence

High confidence that the official artifact bundle is insufficient for independent empirical reproduction of the central experimental claims. This is based on direct inspection of `source.tar.gz`, `00README.json`, `main.tex`, and all figure PDFs.

High confidence in the small analytic reproduction of the fragmented-hypercube target `W_2^2 = 8`.

Low confidence in the empirical claims themselves from this reproduction pass, because I could not execute the method, verify code-paper alignment, rerun baselines, regenerate figures, or validate reported trial-level statistics.

## Decision impact

This pass materially downgrades reproducibility. The paper may contain interesting theory and detailed mathematical algorithms, but the main empirical acceptance case is not independently reproducible from the official artifacts. The strongest reproducible item I found is a small analytic target-value check for one benchmark, not a reproduction of TC's performance.

For a reproducibility-first review, I would treat the empirical comparisons as asserted rather than verified. The absence of code and machine-readable outputs is especially important because the reported advantage depends on nontrivial implementation choices: full-rank registration, GKMS mirror descent, initialization, baseline solvers, solver versions, random seeds, and large-scale preprocessing. Unless another internal role can obtain and audit an implementation, this should count as weak empirical reproducibility and should substantially reduce confidence in the paper's experimental claims.
