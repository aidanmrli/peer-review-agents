# Correctness Specialist Report

Paper ID: 4d7728b5-3db8-4eee-8028-a32080a160b8
Title: Scalable Simulation-Based Model Inference with Test-Time Complexity Control
Assigned role: Correctness Specialist
Date: 2026-04-24

## Scope

I inspected definitions, objectives, evaluation metrics, derivations, and conclusions for technical errors or unsupported inferences. I focused on whether the evidence supports the central claims: joint model/parameter posterior inference, test-time complexity control, scaling to billions of models, and dMRI model selection.

## Evidence examined

- Source LaTeX:
  - `artifacts/source/main.tex`
  - `artifacts/source/appendix.tex`
  - `artifacts/source/references.bib`
- Submitted PDF:
  - `artifacts/paper.pdf`, especially PDF pages 1-7, 14-23, 27, and 32-38.
- Local artifact placeholder:
  - `artifacts/prism/README.md`, which only contains "Tobe published soon." No implementation files were available there.

Commands and checks used:

```bash
curl -fsSL https://koala.science/skill.md
find papers/4d7728b5-3db8-4eee-8028-a32080a160b8 -maxdepth 4 -type f | sort
rg -n "(billions|test-time|complexity|posterior|joint|dMRI|model selection|scalable|RMSE|KSD|ESS|SBC|evidence)" artifacts/source/main.tex artifacts/source/appendix.tex
nl -ba artifacts/source/main.tex | sed -n '150,560p'
nl -ba artifacts/source/appendix.tex | sed -n '1,150p'
nl -ba artifacts/source/appendix.tex | sed -n '150,640p'
nl -ba artifacts/source/appendix.tex | sed -n '709,880p'
nl -ba artifacts/source/appendix.tex | sed -n '1085,1435p'
python -m venv /tmp/agent2-pdfenv && /tmp/agent2-pdfenv/bin/python -m pip install -q pypdf
/tmp/agent2-pdfenv/bin/python - <<'PY'
from pypdf import PdfReader
r = PdfReader("papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/paper.pdf")
print(len(r.pages))
for i, page in enumerate(r.pages, start=1):
    txt = page.extract_text() or ""
    if any(t in txt for t in ["R2", "importance sampling", "Top-5", "dimension-penalized", "NODDI", "SANDI"]):
        print(i, txt[:1000])
PY
```

No OpenReview, citation, social media, or post-publication leakage sources were used. The only external fetch was the required Koala platform skill guide.

## Findings

### 1. Evidence and ESS claims depend on an unstated, likely unavailable density for the diffusion posterior

Candidate error: The paper reports ESS and importance-sampling evidence estimates using weights requiring pointwise evaluation of `q(theta | M, x)`, but the method describes PRISM's parameter posterior as a diffusion sampler and does not explain how the normalized proposal density is evaluated.

Locations:

- Main claim: `artifacts/source/main.tex:464-465`; PDF p. 6 states the model posterior matches "exact evidence-based BMC" with `R2 = 0.97` and that high ESS suggests correction by importance sampling.
- ESS definition: `artifacts/source/appendix.tex:319-326`; PDF pp. 19-20.
- Evidence estimator: `artifacts/source/appendix.tex:343-361`; PDF pp. 20-22.
- Diffusion sampling details: `artifacts/source/appendix.tex:170-186`; PDF pp. 16-17.
- Code artifact absence: `artifacts/prism/README.md`.

Derivation/evidence:

The ESS and evidence equations use weights of the form
`w_i = p(theta_i | x) / q(theta_i | x)` and
`p(x_o | M) approx mean_i p(x_o | M, theta_i) p(theta_i | M) / q(theta_i | M, x_o)`.
These are only computable if the normalized density of the learned diffusion posterior `q` is available at sampled points. The appendix describes an EDM-style diffusion transformer and ODE sampling, but does not describe probability-flow likelihood computation, Jacobian trace estimation, exact change-of-variables density, or any other way to evaluate `q`. KSD does not require `q`, but ESS and the evidence estimator do.

Severity: major.

Consequence: The strongest dMRI model-selection validation, including the "exact evidence-based BMC" comparison and the ESS correction claim, is not reproducible from the stated method. This materially weakens the central dMRI model-selection evidence.

Confidence: high.

### 2. Full dMRI model discovery is presented as broader than the optimization actually performed

Candidate error: The paper defines global model discovery as an argmax over the full combinatorial space, then approximates it by drawing only 100 posterior samples and choosing the highest-probability sampled model.

Locations:

- Extended-model conclusion: `artifacts/source/main.tex:501-508`.
- Model-selection procedure: `artifacts/source/appendix.tex:600-615`; PDF p. 27.
- Result interpretation: `artifacts/source/appendix.tex:618-622`; PDF p. 27.

Derivation/evidence:

For a large posterior over model masks, 100 samples do not solve or reliably approximate `argmax_M q(M | x_o)` unless the MAP model has substantial posterior mass. The paper does not report posterior mass captured by the 100 samples, repeated-sampling stability, missed-mode diagnostics, or comparison to search/beam methods. Restricting the later analysis to the 10 most frequently selected sampled models adds another selection bottleneck.

Severity: major.

Consequence: The evidence supports an exploratory sampling heuristic, not robust global model discovery or full-space model selection. Claims that the method "discovers data-consistent models within large model spaces" should be downgraded or qualified.

Confidence: high.

### 3. dMRI model-space specification has internal inconsistencies affecting the target posterior

Candidate error: The dMRI generative model and priors are not specified consistently enough to define the claimed posterior over the full model family.

Locations:

- Multi-compartment/noise structure: `artifacts/source/appendix.tex:755-770`; PDF p. 31.
- Dimension-penalized product Bernoulli model prior: `artifacts/source/appendix.tex:833-838`; PDF p. 32.
- Watson Zeppelin diffusivity prior: `artifacts/source/appendix.tex:1175-1179`; PDF p. 37.
- Duplicate component IDs and noise IDs in Table 8: `artifacts/source/appendix.tex:1223-1403`; PDF pp. 37-38.

Derivation/evidence:

The text says noise indicators are mutually exclusive, but the later model prior is a product of Bernoulli indicators and does not include a categorical noise constraint. The same table reuses IDs 9 and 10 for NODDI and SANDI components, and ID 10 is also used for Gaussian noise, while Rician noise is ID 11. The Watson Zeppelin prior uses `d_parallel ~ Uniform(0, 3e-9)`, whereas adjacent diffusivity priors use ranges up to `0.01`; with b-values up to 6000 this would make this component nearly unattenuated if interpreted in the same units as the others. The table also permits some Zeppelin variants with `d_perp` not constrained below `d_parallel`, which weakens the orientation interpretation used later for ODF analysis.

Severity: major for reproducible specification of the extended dMRI target; moderate if these are only table transcription errors.

Consequence: The reader cannot unambiguously reconstruct the model family, component indexing, prior over masks/noise, or several component priors. This directly affects the claimed posterior, complexity prior, and extended-space model-selection results.

Confidence: medium-high.

### 4. "Attention-based marginalization" is not a mathematical marginalization result

Candidate error: The method states that masking inactive parameter tokens "effectively marginalizes unused parameters." Masking tokens removes inactive variables from the network input/output, but it is not itself integration over inactive parameters.

Locations:

- Main method: `artifacts/source/main.tex:303`.
- Appendix implementation: `artifacts/source/appendix.tex:153-158`; PDF p. 16.

Derivation/evidence:

Marginalization would require integrating over inactive dimensions under a prior or showing that the target posterior is defined only on active dimensions. The described operation is architectural conditioning on the active mask. It may be a valid active-subspace representation, but the paper does not prove or test that the masked diffusion decoder preserves the correct conditional posterior across models with shared/global parameters.

Severity: moderate.

Consequence: This is mainly a conceptual overclaim, but it matters because the method's core novelty is joint inference across variable-dimensional parameter spaces.

Confidence: medium.

### 5. Lambda conditioning is notationally and conceptually inconsistent for the parameter posterior

Candidate error: The paper alternates between a joint posterior factorization in which `theta` is conditionally independent of `lambda` given `(M, x)` and a decoder/objective conditioned on `lambda`.

Locations:

- Factorization: `artifacts/source/main.tex:260`.
- Approximation targets: `artifacts/source/main.tex:268-270`.
- Parameter decoder: `artifacts/source/main.tex:303`; `artifacts/source/appendix.tex:143`.
- Training loss omits lambda for the parameter loss: `artifacts/source/appendix.tex:221-229`.

Derivation/evidence:

If `lambda` only parameterizes the model prior `p(M | lambda)` and the parameter prior/likelihood are independent of `lambda`, then the exact conditional posterior satisfies `p(theta | M, x, lambda) = p(theta | M, x)`. Conditioning the parameter decoder on `lambda` can be harmless, but it can also introduce spurious lambda dependence unless constrained or tested. The paper does not report such a test.

Severity: moderate.

Consequence: Test-time complexity control should affect model weights, not the model-conditional parameter posterior. The manuscript should either remove lambda conditioning from the parameter target or empirically verify invariance.

Confidence: medium.

### 6. Scaling evidence supports the billion-scale claim only in a restricted sense

Candidate error: The strongest "scales to billions" evidence is for `2^30` models with predictive/calibration metrics, while larger spaces show degradation and model identification is evaluated on small random subspaces.

Locations:

- Abstract and introduction: `artifacts/source/main.tex:162-164`, `artifacts/source/main.tex:202`; PDF pp. 1-2.
- Scaling result: `artifacts/source/main.tex:364-368`; PDF p. 5.
- Appendix scaling metrics: `artifacts/source/appendix.tex:260-284`; PDF pp. 18-19.
- Classification table: `artifacts/source/appendix.tex:294-309`; PDF p. 19.

Derivation/evidence:

The paper reports good rRMSE/rKSD up to `2^30`, but acknowledges clearer deviations at larger scales. Model selection accuracy is measured on a 200-model subspace, not the full model space. At `K=100`, top-1 accuracy is 0.503 and top-5 is 0.905. These results are compatible with approximate predictive amortization, but they do not establish reliable posterior model identification in the full `O(10^30)` space.

Severity: moderate.

Consequence: The abstract's "up to billions" claim is supportable for the `2^30` setting. Stronger language about `O(10^30)` model posterior accuracy or reliable full-space model selection is not supported.

Confidence: high.

### 7. Some metrics are named or interpreted more strongly than their definitions allow

Candidate error: Several metrics are correct as diagnostics but do not justify the stronger posterior-correctness conclusions drawn from them.

Locations:

- Metric caveat: `artifacts/source/appendix.tex:42-46`.
- rRMSE definition: `artifacts/source/appendix.tex:63-65`; PDF p. 14.
- KSD conditional target: `artifacts/source/appendix.tex:104-136`; PDF p. 15.
- dMRI real-data interpretation: `artifacts/source/main.tex:455-465`, `artifacts/source/main.tex:505-508`; PDF pp. 6-7.

Derivation/evidence:

The appendix correctly states that SBC is necessary but not sufficient. The reported "rRMSE" is not a ratio but `RMSE - RMSE_min`, so "relative" is a misleading name. KSD is computed conditional on the generating or selected model and therefore does not validate the joint model posterior. On real dMRI data, KSD/RMSE/LOOCV assess fit under the assumed likelihood family, not anatomical truth or absolute model correctness.

Severity: moderate.

Consequence: The evidence supports useful approximation and fit diagnostics, but not the full strength of statements such as "parameter posterior is near the true posterior" on real data or that selected in-vivo models are scientifically correct rather than statistically favored under the chosen simulator family.

Confidence: high.

## Limitations and blockers

- I did not rerun experiments because the local `prism` artifact contains no implementation beyond a placeholder README.
- I did not use external literature searches or future outcome signals.
- I treated the PDF as the submitted artifact and the LaTeX as the exact source for line-level locations. PDF text extraction was used only for page-level confirmation.

## Final synthesis

I did not find a single algebraic proof error that invalidates the general PRISM idea. The major correctness concern is evidentiary: the dMRI model-selection validation and ESS claims rely on density evaluations for a diffusion posterior that the paper does not define or make reproducible. A second major issue is that full-space dMRI model discovery is only a 100-sample heuristic. The symbolic scaling claim is credible at the `2^30`/billions scale for predictive amortization, but the evidence does not establish reliable full posterior model identification in the much larger spaces. Overall, these issues materially reduce confidence in the central claims, especially dMRI model selection and "exact evidence-based" validation.

Decision impact: substantial downgrade on correctness and reproducibility. The paper may still be promising methodologically, but the current manuscript overstates the support for exact/near-true joint posteriors and robust large-space dMRI model selection.
