# Reproducibility Lead Report

Paper: `a0efc92c-0beb-43e5-a52e-ebeb64b90dfc`  
Title: "Sequence Diffusion Model for Temporal Link Prediction in Continuous-Time Dynamic Graph"  
Role: Reproducibility Lead  

## Claim Being Tested

The paper claims that SDG is an end-to-end sequence diffusion model for continuous-time dynamic graph temporal link prediction and that it achieves state-of-the-art performance on seen and unseen datasets with supporting ablations, robustness, scalability, and uncertainty analysis.

## Team Protocol

I coordinated five independent checks before any public comment:

- Independent Reproducer A: attempted to reproduce the central empirical claims from the paper source, released artifacts, and linked repositories.
- Independent Reproducer B: independently checked table arithmetic, claims against reported values, and whether a runnable SDG path exists.
- Implementation Auditor: inspected bundled source, linked GitHub repositories, evaluators, model entry points, and environment assumptions.
- Correctness Specialist: checked method equations, algorithms, metrics, table-derived conclusions, and ablation claims.
- Literature Specialist: checked the novelty framing against prior work cited by the paper and close permitted prior work.

Reports written:

- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/independent-reproducer-a.md`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/independent-reproducer-b.md`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/implementation-auditor.md`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/correctness-specialist.md`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/literature-specialist.md`

## Artifacts Inspected

Local artifact directory:

- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/paper.pdf`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/source.tar.gz`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq`

Linked repositories cloned for inspection:

- `https://github.com/yule-BUAA/DyGLib.git`, inspected local HEAD `3aacc36b94b8d2d8293d70a74fdf6d39089b4163`
- `https://github.com/TGB-Seq/TGB-Seq.git`, inspected local HEAD `c1ee801ea4301c2f944f5b3da10cbc5310fa4f68`

The LaTeX source states that the code will be available upon acceptance. The provided repositories are upstream baselines, not an SDG implementation.

## Commands and Environment

The checks used the following commands and local environment observations:

```bash
python --version
python - <<'PY'
try:
    import torch
    print(torch.__version__)
except Exception as exc:
    print(type(exc).__name__, exc)
try:
    import numpy
    print(numpy.__version__)
except Exception as exc:
    print(type(exc).__name__, exc)
PY
rg -n -e "SDG|Sequence Diffusion|diffusion|denois|lambda_diff|lambda_inter|Diffusion" artifacts/DyGLib artifacts/TGB-Seq
rg -n -e "model_name|JODIE|DyRep|TGAT|TGN|CAWN|EdgeBank|TCL|GraphMixer|DyGFormer" artifacts/DyGLib artifacts/TGB-Seq
sed -n '1,240p' artifacts/TGB-Seq/tgb_seq/LinkPred/evaluator.py
python train_link_prediction.py --help
```

Environment observations:

- Python: `3.12.12`
- `torch`: not installed in the active environment.
- `numpy`: not installed in the active environment.
- `python train_link_prediction.py --help` failed in the inspected repositories because `tqdm` was missing.

The missing packages prevented even baseline dry-runs in the active environment. More importantly, no SDG implementation, configuration, checkpoint, log, or result-generation script was present, so installing dependencies would not resolve the core reproducibility blocker.

## Independent Reproduction Outcomes

Independent Reproducer A outcome: weak reproducibility. This pass could not run SDG because the released source bundle contains LaTeX and figures, while the linked repositories contain upstream baseline code only. The pass verified that the empirical claim is not independently executable from the provided artifacts.

Independent Reproducer B outcome: weak reproducibility with contradicted table-derived claims. This pass independently recomputed table arithmetic and confirmed that SDG is not best on every reported metric column. It also found a reported average-rank inconsistency in the AP/AUC appendix table and a contradiction between the ablation prose and the ablation table.

The central empirical claim was not reproduced by either independent role. The strongest available check was static table arithmetic, not an executable reproduction.

## Implementation Findings

The implementation audit found the following:

- No SDG model class, diffusion scheduler, denoiser, loss implementation, `lambda_diff`, `lambda_inter`, SDG configuration, checkpoint, training log, or result table generation code exists in the inspected linked repositories.
- The visible model options are baseline methods: JODIE, DyRep, TGAT, TGN, CAWN, EdgeBank, TCL, GraphMixer, and DyGFormer.
- CRAFT is absent from the provided code, despite being a central comparator in the paper.
- The inspected TGB-Seq evaluator path returns MRR and does not trace the reported HR@10 values.
- The paper's implementation description and repository dependency expectations are not enough to recreate SDG training and evaluation.

These are major reproducibility failures because they block the paper's central empirical claim.

## Correctness Findings

The correctness specialist identified several method and reporting issues:

- The reverse diffusion mean in the method and Algorithm 1 is not the standard DDPM posterior mean for x0-prediction. The written coefficient on `x_k` collapses to `sqrt(1 - beta_k)` and omits the required `(1 - bar_alpha_{k-1})/(1 - bar_alpha_k)` factor; the `x_0` coefficient also differs from the standard `sqrt(bar_alpha_{k-1}) beta_k/(1 - bar_alpha_k)`.
- The squared cosine reconstruction loss is not justified as an ELBO variant. Under unit normalization, squared Euclidean distance is proportional to `1 - cosine`, not `(1 - cosine)^2`, and the paper does not define an alternative likelihood that would make the squared cosine term an ELBO.
- Algorithm 2 uses an undefined diffusion timestep `k`.
- The scoring equation is under-specified for candidate-wise ranking and mixes elementwise and matrix scoring notation.
- Complexity accounting appears to omit candidate scoring, the MLP, and the context transformer from the stated dominant terms.

These are major correctness concerns because the paper's proposed generative mechanism and training objective are central to the method.

## Table and Metric Checks

Recomputing the main table values from the LaTeX source gives:

- SDG is best on 16 of 20 main Table 1/2 metric columns, not consistently best across all reported metrics.
- SDG loses on LastFM MRR, LastFM HR@10, UCI HR@10, and YouTube HR@10.
- LastFM MRR: SDG `53.79` vs CRAFT `54.53`, absolute `-0.74`, relative about `-1.36%`.
- LastFM HR@10: SDG `69.15` vs CRAFT `69.95`, absolute `-0.80`, relative about `-1.14%`.
- UCI HR@10: SDG `79.78` vs DyGFormer `82.39`, absolute `-2.61`, relative about `-3.17%`.
- YouTube HR@10: SDG `71.01` vs TGN `71.61`, absolute `-0.60`, relative about `-0.84%`.
- Table 1 Wikipedia HR@10 improvement is miscomputed: `91.40 - 90.95 = 0.45`, relative about `0.49%`, not the reported `0.63` and `0.71%`.
- Table 2 prose claiming HR@10 improvements of `1.59%` to `8.72%` is contradicted by YouTube HR@10.
- The ablation statement that removing any component consistently degrades performance is contradicted by the MLP ablation on Wikipedia MRR and HR@10, where the MLP variant exceeds SDG.

These issues are not formatting only; they directly affect the empirical conclusions.

## Literature Findings

The literature specialist found that a narrow novelty claim may be plausible if framed as end-to-end sequence diffusion for CTDG temporal link ranking. The broad novelty framing is overstated relative to:

- TGB-Seq's sequential temporal link prediction setup and seen/unseen edge evaluation.
- DyGFormer and GraphMixer's historical sequence modeling.
- CRAFT's source-history, temporal encoding, and cross-attention style comparator design.
- CONDA as a close CTDG diffusion prior.
- DiffuRec and PreferDiff as diffusion sequence recommendation baselines relevant to ML-20M, Taobao, and GoogleLocal-style datasets.

The paper's uncertainty discussion is also not supported by calibration, posterior diversity, or probabilistic ranking metrics.

## Final Synthesis and Score Impact

Reproducibility classification: weak reproducibility.

The paper's core empirical claim is not independently reproducible from the provided artifacts by either independent reproducer. The method also contains central equation-level concerns, and several table-derived claims are contradicted by arithmetic from the paper's own reported numbers.

Score impact: this should materially lower confidence in acceptance. I would not credit the claimed state-of-the-art temporal link prediction result without executable SDG code, corrected diffusion equations, traceable HR@10 evaluation, and corrected table/prose claims.
