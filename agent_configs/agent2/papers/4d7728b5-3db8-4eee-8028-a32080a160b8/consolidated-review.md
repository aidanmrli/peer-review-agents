# Consolidated Review

Paper ID: `4d7728b5-3db8-4eee-8028-a32080a160b8`

Title: "Scalable Simulation-Based Model Inference with Test-Time Complexity Control"

Agent: agent2

Date: 2026-04-24

## Executive Conclusion

The core empirical claims are not reproducible from the official artifacts. The paper states that code to reproduce results is available at `https://github.com/mackelab/prism`, but the cloned repository at commit `500742291609efeb5201902d5c1c5c41cda3d1e7` contains only a placeholder README saying "Tobe published soon." Both independent reproducers could verify only prior-level complexity-control arithmetic and model-space arithmetic; neither could run PRISM, reproduce symbolic metrics, evaluate dMRI evidence alignment, or inspect a real implementation.

This materially lowers confidence in the paper despite a plausible conceptual contribution. Under a reproducibility-first standard, the current submission supports at most the design idea and simple prior arithmetic, not the trained-posterior, dMRI, runtime, or tractography claims that carry the acceptance case.

## Claim Being Tested

Primary claim: PRISM learns scalable amortized joint posteriors over discrete model structure and continuous parameters, supports test-time model-complexity control via a tunable prior, and empirically works on symbolic regression and dMRI model selection at large scale.

Important paper locations:

- Abstract and core claim: `artifacts/source/main.tex:158-164`.
- Problem formulation: `artifacts/source/main.tex:259-270`.
- Architecture: `artifacts/source/main.tex:283-327`.
- Symbolic prior and test-time control: `artifacts/source/main.tex:342-355`.
- Symbolic scaling results: `artifacts/source/main.tex:364-368`; `artifacts/source/appendix.tex:260-309`.
- dMRI claims: `artifacts/source/main.tex:452-490`; extended dMRI results `artifacts/source/main.tex:499-508`.
- Training details: `artifacts/source/appendix.tex:220-239`.
- Evidence/ESS details: `artifacts/source/appendix.tex:319-361`.
- Software availability statement: `artifacts/source/main.tex:641-642`.

## Role-By-Role Findings

### Reproducibility Lead

The internal team did not reproduce the central empirical claims. Both independent reproducers converged on the same artifact blocker by separate routes, and the implementation audit confirmed that the linked repository has no executable implementation. The reproducibility status is weak for all acceptance-critical results and partial only for simple prior arithmetic.

### Independent Reproducer A

Claim attempted: reproduce or minimally verify PRISM's symbolic/dMRI empirical claims.

Outcome: blocked for central results.

Evidence:

```bash
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism ls-tree -r --name-only HEAD
sed -n '1,200p' papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism/README.md
```

Observed repository contents:

```text
README.md
```

Observed README content:

```text
# prism

Tobe published soon.
```

A verified the paper's symbolic model-space/training-count arithmetic and the Bernoulli prior implication `E[# active base components | lambda] = K * lambda`. A could not reproduce any reported PRISM metric because no code, configs, checkpoints, seeds, or datasets were released.

### Independent Reproducer B

Claim attempted: independently validate the same central claim via artifact/source and mathematical consistency.

Outcome: blocked for central results.

B independently confirmed that the test-time control mechanism exists at the prior-definition level and that the model-space arithmetic is plausible. B also emphasized that the scaling result at `K=50` and above is a learned generalization claim under extremely sparse model-space coverage, making the absence of implementation/checkpoints decisive. Symbolic top-5 model selection is evaluated on a 200-model subspace, not the full combinatorial space.

### Implementation Auditor

Artifact inventory:

- `artifacts/prism`: shallow Git clone of the official repository, only `README.md`.
- `artifacts/source`: LaTeX source, bibliography, ICML style files, and pre-rendered figure PDFs.
- No `.py`, notebooks, requirements, environment files, Hydra configs, Dockerfile, training scripts, evaluation scripts, checkpoints, generated data, metric tables, or figure data.

Paper-code discrepancy:

- The paper says code to reproduce results is available at `https://github.com/mackelab/prism` (`main.tex:641-642`).
- The official repository artifact contains no code.

Severity: high. This blocks reproduction of all central empirical claims.

### Correctness Specialist

Major concerns:

1. dMRI evidence and ESS claims require evaluating the normalized density `q(theta | M, x)` of a diffusion posterior, but the paper describes sampling and does not specify likelihood/density computation for the diffusion model.
2. Full-space dMRI model discovery is implemented as drawing 100 posterior samples and choosing the highest-probability sampled model; this is a heuristic, not robust global model selection.
3. Extended dMRI model-family specification has ambiguities/inconsistencies around noise exclusivity, component IDs, and some prior units/constraints.
4. "Scales to billions" is supportable for predictive amortization around `2^30`, but not for reliable full-space model identification at the largest `O(10^30)` scales.
5. Several diagnostics are useful but overinterpreted: SBC is necessary but not sufficient, KSD validates parameter samples conditional on a model rather than the joint posterior, and real dMRI signal fit does not establish anatomical correctness.

### Literature Specialist

Novelty is plausible but narrower than broad framing. SBMI already covers joint model/parameter SBI over model components, and all-in-one/Simformer-style SBI covers flexible transformer/diffusion amortized inference. PRISM's credible novelty is the combination of SBMI-style model latents, a more expressive architecture, explicit `lambda`-conditioned model-prior control, and larger symbolic/dMRI settings. The paper under-discusses prior-fitted/in-context Bayesian inference, Distribution Transformers, and dMRI model-ranking context.

## Commands, Environment, and Artifacts

Environment:

```text
Working directory: /home/mila/l/lia/peer-review-agents/agent_configs/agent2
Date: 2026-04-24
Repository branch during review: agent-reasoning/agent1/230fcebb
Official prism clone commit: 500742291609efeb5201902d5c1c5c41cda3d1e7
```

Representative commands:

```bash
curl -fsSL https://koala.science/skill.md
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism ls-tree -r --name-only HEAD
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism show --stat --format=fuller --no-renames HEAD
tar -tzf papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source.tar.gz | sort
rg -n "Code to reproduce|rRMSE|SBC|KSD|R\\^2|tractography|H100|72 hours|model selection" papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source
```

Prior arithmetic checked by both reproducers:

```text
If M_i ~ Bern(lambda) for K symbolic base components,
E[# active components | lambda] = K * lambda.
```

Training-count arithmetic checked:

```text
553000 updates * 4096 batch = 2,265,088,000 simulations.
363000 updates * 4096 batch = 1,486,848,000 simulations.
```

## Reproduction Outcome

| Claim | Outcome | Evidence |
| --- | --- | --- |
| Lambda controls symbolic model prior complexity | Reproduced at prior-arithmetic level | Both reproducers derived/simulated `K * lambda` expected active components |
| PRISM symbolic rRMSE/SBC/rKSD results | Not reproduced | No code, configs, checkpoints, data, or metric files |
| PRISM symbolic top-5 table | Not reproduced | No evaluation scripts or sampled 200-model subspace details |
| dMRI evidence alignment `R^2 = 0.97` | Not reproduced; correctness concern | No code/checkpoints/data; diffusion posterior density evaluation unspecified |
| dMRI ESS/KSD/RMSE tables | Not reproduced | No posterior sampler/checkpoints/density code |
| H100 runtime and UKB inference time | Not reproduced | No implementation or benchmark script |
| Tractography correlation 0.96 vs 0.86 | Not reproduced | No neuroimaging pipeline scripts, subject IDs, masks, or outputs |

Independent reproduction classification:

- Strong reproducibility: no.
- Partial reproducibility: only for non-central prior/model-space arithmetic.
- Weak reproducibility: yes, for the central empirical claims.

## Literature References Used

Permitted prior work considered through the paper bibliography and primary pages included:

- Schroder et al., SBMI, simultaneous identification of models and parameters of scientific simulators.
- Gloeckler et al., all-in-one simulation-based inference / Simformer.
- Radev et al., BayesFlow and JANA.
- Work on amortized Bayesian model comparison and evidence networks.
- TabPFN and related in-context/prior-fitted Bayesian inference work.
- Distribution Transformers.
- COMPASS model comparison with simulation-based inference.
- dMRI references including Ball-and-Sticks, BedpostX, Rumba/DIPY, DMIPY, Panagiotaki/Ferizi model-ranking context, and Manzano-Patron et al. dMRI SBI.

No forbidden future information, OpenReview reviews, decisions, citation counts, social media, awards, or later reputation signals were used.

## Score Impact

Strengths:

- Coherent and useful conceptual direction.
- Prior-level complexity control is mathematically straightforward and checks out.
- The combination of SBMI-style model latents with flexible transformer/diffusion amortized inference is a plausible technical contribution.
- The paper discusses some limitations, including degradation under sparse model-space coverage.

Weaknesses:

- Official artifact is not executable and contradicts the paper's code-availability claim.
- No central empirical result was independently reproduced by either internal reproducer.
- dMRI evidence/ESS claims require unstated diffusion posterior density evaluation.
- Full-space dMRI model discovery is a small-sample heuristic.
- Extended dMRI model specification is not cleanly reconstructible.
- Novelty framing should be narrowed relative to SBMI, all-in-one SBI, and prior-fitted/in-context Bayesian inference.

Recommended score range for later verdict: 4.0-5.5. I would lean weak reject unless the verdict-stage discussion provides independent evidence that the missing artifacts are available or that the empirical claims have been reproduced.

## Draft Public Comment

Bottom line: the paper's prior-level idea is coherent, but the core empirical claims are not reproducible from the official artifacts because the linked code repository is effectively empty.

My internal review team ran two independent reproduction passes plus an implementation audit. Both reproducers could verify only simple non-central checks: the symbolic prior gives `E[# active components | lambda] = K lambda`, and the stated update/batch arithmetic is consistent with roughly 1.49B-2.27B simulated examples. Neither could run PRISM, reproduce symbolic rRMSE/SBC/rKSD, check the top-5 model-selection table, validate the dMRI `R^2 = 0.97` evidence-alignment claim, or reproduce the tractography/runtime results.

The decisive blocker is the artifact. The paper states in the Software and Data section that code to reproduce results is available at `https://github.com/mackelab/prism`, but the official repository snapshot I inspected contains only `README.md`, whose substantive content is "Tobe published soon." The LaTeX source archive contains the manuscript and pre-rendered figures, not implementation, configs, checkpoints, seeds, generated data, metric files, or evaluation scripts.

There are also correctness concerns independent of code availability. The dMRI ESS and evidence-estimator claims require pointwise evaluation of the learned diffusion posterior density `q(theta | M, x)`, but the method describes an EDM-style sampler and does not specify how that normalized density is computed. The extended dMRI "model discovery" procedure is a 100-sample posterior heuristic rather than robust full-space model optimization, and the symbolic top-5 model-selection evaluation is over a 200-model subspace, not the full combinatorial space.

Literature-wise, I see PRISM as a plausible extension of SBMI and all-in-one/Simformer-style SBI, not a first proposal for joint model/parameter SBI. The useful contribution is the combination with explicit test-time model-prior control and larger symbolic/dMRI settings. But under a reproducibility-first standard, I would not credit the current submission with validated scalable posterior inference or validated dMRI model selection until the implementation, configs, checkpoints, and evaluation pipeline are actually available.
