# Independent Reproducer B Report

## Paper ID and Title

- Paper ID: `4d7728b5-3db8-4eee-8028-a32080a160b8`
- Title: "Scalable Simulation-Based Model Inference with Test-Time Complexity Control"

## Assigned Role

Independent Reproducer B. I used an artifact/source and mathematical consistency route, not Reproducer A's report or notes. I did not inspect `independent-reproducer-a.md`, OpenReview, citation counts, social media, acceptance signals, or any non-artifact future/leakage source.

## Task Scope

Independently validate the central claim through an alternative route: whether the official paper/source/artifacts support the claim that PRISM performs scalable joint model-parameter inference over combinatorial simulator families while allowing test-time complexity control through a tunable model prior.

## Evidence Examined

- Paper PDF: `artifacts/paper.pdf` (direct text extraction unavailable because `pdftotext` is not installed).
- LaTeX source: `artifacts/source/main.tex`, `artifacts/source/appendix.tex`, `artifacts/source/00README.json`.
- Source archive: `artifacts/source.tar.gz`.
- Official repository snapshot: `artifacts/prism`.
- Figure/table assets under `artifacts/source/figures/`.

Key source locations:

- Abstract claim: `main.tex:158-164` says PRISM infers joint model/parameter posteriors, enables test-time complexity control, and scales to combinatorially many model instantiations.
- Symbolic model/prior: `main.tex:342-351` defines additive components and the Bernoulli/categorical model prior with lambda.
- Symbolic scaling claims: `main.tex:364-368` and `appendix.tex:260-284`.
- Training details and compute: `appendix.tex:220-239`.
- Architecture and complexity: `appendix.tex:142-168`, `appendix.tex:170-186`.
- dMRI evidence-estimation claim: `appendix.tex:343-361`.
- Software availability statement: `main.tex:641-642`; artifact repository README says only "Tobe published soon."

## Commands and Derivations

Artifact inventory:

```bash
find papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts -maxdepth 3 -type f -printf '%p\n' | sort
tar -tzf papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source.tar.gz | sort
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism ls-tree -r --name-only HEAD
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism log --oneline -3
du -ah papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts | sort -h | tail -30
```

Observed:

- `artifacts/prism` has only one tracked file, `README.md`, at commit `5007422 Update README.md`.
- `artifacts/prism/README.md` contains only:

```text
# prism

Tobe published soon.
```

- The source tarball contains LaTeX, bibliography/style files, and figure PDFs. I found no runnable training/evaluation code, no Hydra configs, no seeds, no checkpoints, no generated datasets, no metric CSV/JSON files, and no script that can recompute the reported tables/figures.

Model-space arithmetic:

```bash
awk 'BEGIN{printf "%5s %15s %18s %18s %18s\n", "K", "noise choices", "2^K", "2^K*noise", "coverage 2.265B"; for(i=1;i<=5;i++){K=(i==1?15:(i==2?30:(i==3?50:(i==4?80:100)))); n=(K==15?5:8); space=(2^K)*n; cov=2265088000/space; printf "%5d %15d %18.6g %18.6g %17.6g%%\n", K,n,2^K,space,cov*100}}'
```

Output:

```text
    K   noise choices                2^K          2^K*noise    coverage 2.265B
   15               5              32768             163840        1.3825e+06%
   30               8        1.07374e+09        8.58993e+09           26.3691%
   50               8         1.1259e+15         9.0072e+15       2.51475e-05%
   80               8        1.20893e+24        9.67141e+24       2.34205e-14%
  100               8        1.26765e+30        1.01412e+31       2.23355e-20%
```

Complexity-control derivation for the symbolic task:

- Source prior: `p(M | lambda) = prod_k Ber(M_k, lambda) * Cat(noise)` with mutually exclusive noise.
- Therefore, for K base components, `E[# active base components | lambda] = K * lambda` and `Var[# active base components | lambda] = K * lambda * (1 - lambda)`.

```bash
awk 'BEGIN{for(lambda=0; lambda<=1.0001; lambda+=0.25){printf "lambda=%.2f: E[active base components] K=50 -> %.2f; K=100 -> %.2f\n", lambda, 50*lambda, 100*lambda}}'
```

Output:

```text
lambda=0.00: E[active base components] K=50 -> 0.00; K=100 -> 0.00
lambda=0.25: E[active base components] K=50 -> 12.50; K=100 -> 25.00
lambda=0.50: E[active base components] K=50 -> 25.00; K=100 -> 50.00
lambda=0.75: E[active base components] K=50 -> 37.50; K=100 -> 75.00
lambda=1.00: E[active base components] K=50 -> 50.00; K=100 -> 100.00
```

Training-coverage check:

- Appendix says K=50 symbolic runs used 553k update steps for the smallest network and about 363k for the largest, with batch size 4096 (`appendix.tex:237`, `appendix.tex:263`).
- This corresponds to `553000 * 4096 = 2.265088e9` and `363000 * 4096 = 1.486848e9` simulated examples.
- If the full symbolic model count includes the 8 mutually exclusive noise choices, K=50 has `2^50 * 8 = 9.007e15` configurations, so the upper-bound coverage is only about `2.5e-5%`.
- If one ignores the noise categorical, coverage is about `2.0e-4%`, closer to the paper's "about 10^-4%" statement. The same ambiguity matters for K=30: including noise gives about 0.26 simulations per configuration; ignoring noise gives about 2.1, matching the paper's "roughly two simulations per model configuration."

## Findings

1. The mathematical mechanism for test-time complexity control is structurally valid in the source. For the symbolic task, lambda directly controls the Bernoulli inclusion probability, so expected active base components scale linearly with lambda. For the dMRI extension, the dimension-penalized prior in `appendix.tex:833-838` similarly makes higher-dimensional components less likely when lambda is small. This supports the existence of a tunable prior, at least at the prior-definition level.

2. The combinatorial scale claim is arithmetically plausible. With K=30 and eight noise choices, the symbolic family has about 8.6 billion configurations; K=100 gives about 1.0e31 configurations. This is consistent with `main.tex:364` describing spaces up to O(10^30), and stronger than the abstract's "up to billions" phrasing.

3. The empirical central claim is not independently reproducible from the released artifacts. The paper claims rRMSE near the noise floor up to 2^30, calibration near expected Monte Carlo error, high top-5 classification accuracy, dMRI model-posterior alignment with importance-sampling evidence (R^2=0.97), and runtime/ESS results. The official artifact set contains no executable code, no configs, no checkpoints, no seeds, no data, and no metric files. The linked repository snapshot contradicts the software availability claim because it contains only a placeholder README.

4. The reported scaling success necessarily relies on extreme extrapolation, especially at K=50 and above. The appendix admits this qualitatively. My arithmetic confirms that even under a maximally generous no-repeat assumption, K=50 coverage is negligible relative to the full model family. That does not invalidate the result, but it makes the missing code/checkpoints particularly serious: the core result is a learned generalization claim, not a derivation.

5. The model-selection evaluation is narrower than the headline may suggest. The symbolic top-5 classification result is computed on a random 200-model subspace, not the full combinatorial space (`appendix.tex:274-284`). At K=100, reported top-1 accuracy is 0.503 and macro F1 is 0.463, while top-5 remains 0.905 (`appendix.tex:294-305`). This supports posterior concentration over a small candidate set, but not exact large-space model identification.

6. The dMRI evidence-validation claim is methodologically plausible but unauditable from artifacts. The source describes an importance-sampling evidence estimator using samples from q and reports R^2=0.99 between two MC estimators and R^2=0.97 between q/p-prior proxy and MC evidence (`appendix.tex:343-361`). Without the code, q-density evaluation, data subset, random seeds, and checkpoints, I cannot verify support conditions, numerical stability, or reproduce the correlation.

## Limitations and Blockers

- Full empirical reproduction is blocked by absence of released implementation and experiment artifacts. The artifact repository is effectively empty.
- I could not use `pdftotext`; I relied on the LaTeX source as the text-equivalent official artifact.
- Figure PDFs appear to be final plots, not underlying data tables. I did not attempt to reverse-engineer numeric series from rendered plots because that would not reproduce the experiments.
- I did not compare against Reproducer A because the task required independence and I did not read A's report.

## Confidence

- High confidence in the artifact audit and arithmetic checks.
- Medium confidence in the mathematical conclusion that lambda implements test-time prior complexity control at the prior-definition level.
- Low confidence in the empirical performance claims as independently reproduced, because no runnable code/data/checkpoints were available.

## Decision Impact

This is a material reproducibility weakness. The paper's conceptual mechanism and model-space arithmetic check out, but the acceptance-relevant empirical claims are not reproducible from the official artifacts. I would mark the central claim as only partially validated: prior-level complexity control is supported; scalable high-quality inference over huge model spaces and the dMRI evidence-alignment results remain unverified. This should materially lower confidence unless another reviewer obtains runnable artifacts or independently reproduces the experiments by reimplementing the method.
