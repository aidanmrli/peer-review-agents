# Correctness Specialist Report

Paper: `a0efc92c-0beb-43e5-a52e-ebeb64b90dfc`  
Title: "Sequence Diffusion Model for Temporal Link Prediction in Continuous-Time Dynamic Graph"  
Role: Correctness Specialist  

## Scope

I inspected `artifacts/example_paper.tex`, the appendix algorithms, table-derived conclusions, metric definitions, time-complexity claims, and the bundled TGB-Seq/DyGLib evaluator and plotting artifacts. I recomputed the relative and absolute improvements in Tables 1 and 2 and the ablation deltas where the table values were available.

Commands and checks used:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,260p' skills/correctness-specialist.md
rg -n -e '\\section' -e '\\subsection' -e '\\begin\{table' -e '\\begin\{algorithm' -e 'MRR' -e 'HR@10' -e 'Rel.Imprv' -e 'Abs.Imprv' -e 'ablation' -e 'complex' -e 'ELBO' artifacts/example_paper.tex
nl -ba artifacts/example_paper.tex | sed -n '149,180p'
nl -ba artifacts/example_paper.tex | sed -n '249,292p'
nl -ba artifacts/example_paper.tex | sed -n '294,427p'
nl -ba artifacts/example_paper.tex | sed -n '708,753p'
nl -ba artifacts/example_paper.tex | sed -n '921,966p'
nl -ba artifacts/example_paper.tex | sed -n '997,1033p'
sed -n '1,240p' artifacts/TGB-Seq/tgb_seq/LinkPred/evaluator.py
sed -n '1,300p' artifacts/TGB-Seq/examples/evaluate_models_utils_mrr.py
rg -n --glob '!**/logos/*.svg' --glob '!**/poetry.lock' 'SDG|Sequence Diffusion|diffusion|denoise|lambda_diff|lambda_inter|alpha_bar|alphas_cumprod|beta' artifacts/TGB-Seq artifacts/DyGLib
python - <<'PY'
# hard-coded table recomputation from Tables 1, 2, and ablation table
PY
```

`pdftotext` was not installed, so figure-only claims in `robustness.pdf`, `scalability.pdf`, `hyperparameters.pdf`, `noise_schedule.pdf`, and `nl.pdf` could not be exactly extracted from the PDFs. The available plot scripts do not contain the SDG figure data for these figures.

## Paper Claim Being Tested

The central claim is that SDG is a technically valid sequence-level DDPM-style model for temporal link prediction and that the reported experiments show consistent state-of-the-art performance, meaningful ablation support, and favorable complexity/scalability.

## Fatal Issues

No fatal correctness issue is fully established from the text alone because the actual SDG implementation is not present in the bundled code directories. However, the major issues below are central enough that the method and empirical claims should be materially discounted unless the authors provide corrected equations, corrected claims, and executable code showing that the experiments used the intended formulas.

## Major Issues

### 1. The reverse diffusion mean is mathematically wrong

- Candidate error: The DDPM posterior mean used for x0-prediction is not the standard reverse mean.
- Location: `example_paper.tex` lines 170-173, repeated in Algorithm 1 at lines 1008-1011.
- Why it is wrong: For DDPM with `alpha_k = 1 - beta_k` and `bar_alpha_k = prod_{s=1}^k alpha_s`, the posterior mean is

```text
mu_tilde_k(x_k, x_0) =
  [sqrt(bar_alpha_{k-1}) beta_k / (1 - bar_alpha_k)] x_0
  + [sqrt(alpha_k) (1 - bar_alpha_{k-1}) / (1 - bar_alpha_k)] x_k.
```

The paper writes

```text
sqrt(1 - beta^k) * (1 - bar_alpha^k)/(1 - bar_alpha^k) * x^k
+ alpha^{k-1} beta^k/(1 - bar_alpha^k) * x_hat^0,
```

which simplifies the first coefficient to `sqrt(1 - beta^k)` and omits the required `(1 - bar_alpha_{k-1})/(1 - bar_alpha_k)` factor. The second coefficient also uses `alpha^{k-1}` rather than `sqrt(bar_alpha_{k-1})`.
- Evidence or derivation: The standard DDPM posterior is derived from `q(x_k | x_0)` and `q(x_k | x_{k-1})`; the posterior variance-normalized product of Gaussians produces the coefficients above. The paper's expression is not algebraically equivalent except in degenerate cases.
- Severity: Major.
- Consequence for acceptance: The algorithm as written does not implement a valid DDPM reverse sampler. Because inference in SDG depends on repeated reverse steps, this directly undermines the stated generative mechanism and Algorithm 1. If the implementation used the correct formula, the paper must be corrected; if it used the written formula, the reported method is technically unsound.

### 2. The cosine reconstruction loss is not justified as a valid ELBO variant

- Candidate error: The paper claims the squared cosine reconstruction loss corresponds to a valid ELBO variant, but the appendix only derives MSE and then asserts an unsupported replacement.
- Location: Main text lines 253-259; appendix lines 921-966.
- Why it is wrong: The ELBO derivation for a Gaussian DDPM yields a weighted squared Euclidean error between posterior means, or equivalently an x0 MSE under fixed-variance Gaussian assumptions. The appendix shows that, under unit-normalized embeddings, `||x - y||_2^2 = 2(1 - cos(x,y))`. That does not justify the actual loss in lines 255-256 and 958-965:

```text
(1 - cos(x,y))^2.
```

Even under unit norm, this squared cosine error is proportional to `||x - y||_2^4`, not to the Gaussian ELBO MSE term. The paper also does not establish that `X^0` and `X_hat^0` are normalized during training, and it does not define an alternative likelihood, such as a directional distribution, under which this is an ELBO.
- Evidence or derivation:

```text
If ||x|| = ||y|| = 1:
MSE = ||x-y||^2 = 2(1-cos(x,y)).
Paper loss = (1-cos(x,y))^2 = MSE^2 / 4.
```

Minimizing the two losses may share minimizers in an unconstrained toy case, but the objective is not the same ELBO term and has different gradients and weighting.
- Severity: Major.
- Consequence for acceptance: The paper's theoretical support for the diffusion objective is invalid. The loss may still be a heuristic, but it is not established as a valid ELBO variant.

### 3. The main empirical conclusion falsely says SDG is best across all unseen datasets

- Candidate error: The text says SDG consistently achieves the best performance across all five TGB-Seq datasets and improves HR@10 by `1.59%-8.72%`, but Table 2 shows a negative HR@10 improvement on YouTube.
- Location: Table 2 lines 330-366; conclusion text line 401.
- Why it is wrong: On YouTube HR@10, TGN has `71.61`, while SDG has `71.01`. The table correctly reports `Rel.Imprv. = -0.84%` and `Abs.Imprv. = -0.60`, but the prose claims uniformly best performance and a positive HR@10 improvement range.
- Evidence or derivation:

```text
YouTube HR@10:
SDG = 71.01
Best baseline = TGN = 71.61
Absolute improvement = 71.01 - 71.61 = -0.60
Relative improvement = -0.60 / 71.61 * 100 = -0.838%
```

- Severity: Major.
- Consequence for acceptance: This is a direct overstatement of a central result. The paper should say SDG is best on 9 of 10 Table 2 metrics, not all metrics, and should not report a strictly positive HR@10 improvement range.

### 4. The ablation conclusion overstates component necessity

- Candidate error: The paper claims that removing any SDG component "consistently degrades performance," but the MLP denoiser variant beats SDG on Wikipedia.
- Location: Ablation text lines 405-407; Table 3 lines 416-424.
- Why it is wrong: In Table 3, the MLP variant has Wikipedia MRR `89.40` and HR@10 `91.69`, both higher than SDG's `89.16` and `91.45`. Therefore, replacing the cross-attention denoising network with an MLP does not consistently degrade performance.
- Evidence or derivation:

```text
Wikipedia MRR: MLP - SDG = 89.40 - 89.16 = +0.24
Wikipedia HR@10: MLP - SDG = 91.69 - 91.45 = +0.24
```

Other ablation drops are very small on Wikipedia and Reddit, e.g. w/o Diff is only `0.05` MRR and `0.04` HR@10 below SDG on Wikipedia.
- Severity: Major.
- Consequence for acceptance: The ablation table supports strong effects on GoogleLocal and YouTube, but it does not support the universal claim that every design choice is consistently necessary across datasets.

## Moderate Issues

### 5. Wikipedia HR@10 improvement in Table 1 is miscomputed

- Candidate error: The relative and absolute improvements for Wikipedia HR@10 in Table 1 do not match the table values.
- Location: Table 1 lines 311-317.
- Why it is wrong: The best baseline for Wikipedia HR@10 is DyGFormer at `90.95`, and SDG is `91.40`.
- Evidence or derivation:

```text
Absolute improvement = 91.40 - 90.95 = 0.45
Relative improvement = 0.45 / 90.95 * 100 = 0.495%
```

The paper reports `Abs.Imprv. = 0.63` and `Rel.Imprv. = 0.71%`.
- Severity: Moderate.
- Consequence for acceptance: This does not change the ranking, but it shows table-derived conclusions were not consistently audited.

### 6. The method's scoring equation is internally inconsistent and potentially leakage-prone as written

- Candidate error: Equation 9 and its explanation mix element-wise product, dot product, full-candidate scoring, and target-sequence scoring in incompatible shapes.
- Location: Lines 261-266 and inference text line 324.
- Why it is wrong or unsupported:
  - The text says "element-wise product" but line 266 says `\cdot` denotes a dot product.
  - `X^0` and `H(T_{u,t})` are sequence embeddings in `R^{L x d}`. A dot product over `d` would yield one scalar per position, whereas the paper says `y_t in R^{L x N}` and that each row scores all candidate nodes.
  - During training, line 266 says the candidate set sequence contains one positive and one negative sequence, which conflicts with the `R^{L x N}` all-candidate score matrix.
  - The formula uses `H(T_{u,t})`, the target sequence containing the true destination, while inference must score arbitrary candidates from `C_{u,t}`. The paper does not precisely define how candidate-specific elapsed time features and candidate embeddings are combined with the generated last-position embedding.
- Evidence or derivation: The dimensions do not type-check under a single interpretation:

```text
X^0: L x d
H(T_{u,t}): L x d for one target sequence, or N x d for all candidates
dot(X^0, H(T)): L or L x N depending on interpretation
MLP(concat(dot, gamma(delta_t))): unspecified because gamma(delta_t) is d-dimensional
claimed y_t: L x N
```

- Severity: Moderate.
- Consequence for acceptance: The scoring rule is central to converting generated embeddings into ranks. As written, it is under-specified enough that independent implementation could choose materially different candidate scoring behavior.

### 7. Algorithm 2 uses an undefined diffusion timestep

- Candidate error: The training algorithm says to add Gaussian noise at timestep `k`, but never samples or defines `k`.
- Location: Lines 999 and 1024-1028.
- Why it is wrong: The prose says training uses a randomly chosen diffusion step, but Algorithm 2 samples only epsilon and then uses `k` in line 1026. A correct algorithm must sample `k`, usually uniformly from `{1,...,K}` or from a stated schedule.
- Evidence or derivation: Algorithm 2 has no line equivalent to `k ~ Uniform({1,...,K})`.
- Severity: Moderate.
- Consequence for acceptance: This is probably a presentation error, but it makes the formal training algorithm non-executable as written.

### 8. The time-complexity table undercounts or obscures SDG scoring costs

- Candidate error: The SDG compute likelihood complexity is reported as `K B (L^2 d + L d^2) + B M d`, but Equation 9 describes an MLP over candidate-specific concatenated features and the method also computes a context transformer.
- Location: Complexity table and explanation lines 708-753; scoring Equation 9 lines 261-266.
- Why it is wrong or unsupported:
  - If candidate scoring uses an MLP with hidden width proportional to `d`, the candidate term should include at least `O(B M d^2)` or a stated smaller architecture, not only `O(B M d)`.
  - If scores are produced for each sequence position and candidate as claimed (`L x N`), a candidate-position term such as `O(B M L d)` is also plausible.
  - The context Transformer over `S_{u,t}` adds `O(B(L^2 d + L d^2))` before denoising. This is dominated by the `K` denoising term only when `K > 1`, but should still be stated for a fair derivation.
  - The table uses `N_avg` for SDG extraction and `N_deg` for other methods without defining a meaningful difference.
- Evidence or derivation: Equation 9 invokes `MLP(concat(...))`, not a pure dot product. A one-hidden-layer MLP on `d`-dimensional candidate features costs `Theta(d^2)` per candidate unless the hidden dimension is constant or explicitly smaller.
- Severity: Moderate.
- Consequence for acceptance: The efficiency claim is plausible only after implementation-level clarification. The written complexity table is not a reliable basis for the "favorable trade-off" conclusion.

### 9. Metric definitions are incomplete for HR@10 and tie handling

- Candidate error: The paper defines MRR and HR@10 only verbally and does not specify rank tie handling, candidate pool size, or HR@10 computation for the reported TGB-Seq tables.
- Location: Evaluation protocol lines 378-380; bundled evaluator `artifacts/TGB-Seq/tgb_seq/LinkPred/evaluator.py`.
- Why it is wrong or unsupported: The TGB-Seq evaluator artifact returns only MRR using an averaged optimistic/pessimistic rank under ties. It does not compute HR@10. The paper reports HR@10 throughout Tables 1 and 2, but the exact HR@10 computation and tie convention are not specified in the text or evaluator artifact inspected.
- Evidence or derivation: `Evaluator.eval` computes:

```python
optimistic_rank = (y_pred_neg > y_pred_pos).sum(axis=1)
pessimistic_rank = (y_pred_neg >= y_pred_pos).sum(axis=1)
ranking_list = 0.5 * (optimistic_rank + pessimistic_rank) + 1
mrr_list = 1. / ranking_list.astype(np.float32)
return mrr_list
```

No HR@10 is returned there.
- Severity: Moderate.
- Consequence for acceptance: HR@10 is one of the two main metrics. The results may be valid, but the paper does not provide enough metric detail for an unambiguous independent implementation.

## Minor Issues

### 10. Several percentage statements use inconsistent rounding or denominator conventions

- Candidate error: The prose says SDG underperforms on LastFM and UCI HR@10 by `1.10%` and `3.27%`.
- Location: Line 399.
- Why it is wrong or unsupported: Using the best baseline as denominator gives:

```text
LastFM HR@10: (69.15 - 69.95) / 69.95 * 100 = -1.144%
UCI HR@10: (79.78 - 82.39) / 82.39 * 100 = -3.168%
```

These correspond to about `1.14%` and `3.17%`, not `1.10%` and `3.27%`. The table itself reports `-1.14%` and `-3.16%`.
- Severity: Minor.
- Consequence for acceptance: The numerical discrepancy is small, but it reinforces that the empirical prose was not carefully aligned with the tables.

### 11. The bundled code artifacts do not contain the SDG implementation

- Candidate error: The artifacts include TGB-Seq and DyGLib code, but no SDG, diffusion, denoising, or `lambda_diff` implementation was found.
- Location: `artifacts/TGB-Seq`, `artifacts/DyGLib`; paper line 328 says code will be available upon acceptance.
- Why it is wrong or unsupported: The absence of SDG code prevents checking whether the experiments used the written equations or corrected private code. This is primarily a reproducibility limitation, but it also blocks resolution of the correctness issues above.
- Evidence or derivation:

```bash
rg -n --glob '!**/logos/*.svg' --glob '!**/poetry.lock' \
  'SDG|Sequence Diffusion|diffusion|denoise|lambda_diff|lambda_inter|alpha_bar|alphas_cumprod|beta' \
  artifacts/TGB-Seq artifacts/DyGLib
```

returned no matches.
- Severity: Minor as a correctness finding, major as a reproducibility limitation.
- Consequence for acceptance: The paper cannot rely on private implementation behavior to repair the public math and algorithm. The public artifact does not let reviewers adjudicate whether the reported results used the algorithm in the paper.

## Recomputed Table Improvements

Formula used: `relative = (SDG - best_baseline) / best_baseline * 100`; `absolute = SDG - best_baseline`. Negative values mean SDG is below the best baseline.

Table 1, seen-dominant datasets:

| Metric | Recomputed rel. | Reported rel. | Recomputed abs. | Reported abs. | Status |
|---|---:|---:|---:|---:|---|
| Wikipedia MRR | 0.405% | 0.41% | 0.36 | 0.37 | rounding only |
| Wikipedia HR@10 | 0.495% | 0.71% | 0.45 | 0.63 | incorrect |
| Reddit MRR | 0.202% | 0.20% | 0.18 | 0.18 | ok |
| Reddit HR@10 | 0.392% | 0.39% | 0.37 | 0.37 | ok |
| MOOC MRR | 2.994% | 2.99% | 1.76 | 1.76 | ok |
| MOOC HR@10 | 1.589% | 1.59% | 1.25 | 1.25 | ok |
| LastFM MRR | -1.357% | -1.36% | -0.74 | -0.74 | ok |
| LastFM HR@10 | -1.144% | -1.14% | -0.80 | -0.80 | ok |
| UCI MRR | 0.528% | 0.53% | 0.40 | 0.40 | ok |
| UCI HR@10 | -3.168% | -3.16% | -2.61 | -2.61 | ok |

Table 2, unseen-dominant datasets:

| Metric | Recomputed rel. | Reported rel. | Recomputed abs. | Reported abs. | Status |
|---|---:|---:|---:|---:|---|
| GoogleLocal MRR | 14.484% | 14.48% | 7.92 | 7.92 | ok |
| GoogleLocal HR@10 | 8.721% | 8.72% | 6.30 | 6.30 | ok |
| YouTube MRR | 2.697% | 2.70% | 1.59 | 1.59 | ok |
| YouTube HR@10 | -0.838% | -0.84% | -0.60 | -0.60 | ok, contradicts prose |
| Flickr MRR | 0.849% | 0.84% | 0.52 | 0.52 | rounding only |
| Flickr HR@10 | 2.288% | 2.29% | 1.81 | 1.81 | ok |
| ML-20M MRR | 1.722% | 1.72% | 0.62 | 0.62 | ok |
| ML-20M HR@10 | 2.429% | 2.43% | 1.27 | 1.27 | ok |
| Taobao MRR | 3.397% | 3.40% | 2.29 | 2.29 | ok |
| Taobao HR@10 | 2.972% | 2.97% | 2.35 | 2.35 | ok |

Key ablation deltas from Table 3:

| Dataset/metric | Variant | SDG - variant | Relative drop vs SDG |
|---|---|---:|---:|
| Wikipedia MRR | MLP | -0.24 | -0.27% |
| Wikipedia HR@10 | MLP | -0.24 | -0.26% |
| GoogleLocal MRR | MSE | 7.71 | 12.32% |
| GoogleLocal MRR | w/o Seq | 7.33 | 11.71% |
| GoogleLocal MRR | w/o Diff | 6.93 | 11.07% |
| YouTube MRR | MLP | 8.63 | 14.32% |

The GoogleLocal "MSE degrades by 12.32%" claim is arithmetically correct if measured as `(SDG - MSE) / SDG`. The universal ablation conclusion is not correct because MLP improves Wikipedia.

## Final Synthesis and Score Impact

The paper has several correctness problems that are decision-relevant. The most serious are the incorrect DDPM reverse mean, the invalid ELBO justification for the squared cosine loss, and empirical prose that asserts uniform best performance despite table entries showing negative HR@10 improvements. The ablation and complexity sections also overstate what is supported by their own tables and equations.

I would materially downgrade the paper on correctness. The empirical tables still show meaningful gains on several datasets, especially GoogleLocal and some large-scale settings, but the theoretical framing and several central conclusions require correction before the claims can be trusted.
