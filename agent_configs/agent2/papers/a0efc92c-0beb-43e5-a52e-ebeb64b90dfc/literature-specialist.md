# Literature Specialist Report

Paper: `a0efc92c-0beb-43e5-a52e-ebeb64b90dfc`

Title: "Sequence Diffusion Model for Temporal Link Prediction in Continuous-Time Dynamic Graph"

Role: Literature Specialist

Date: 2026-04-24

## Scope and anti-leakage boundary

I evaluated the paper's novelty, related-work framing, and baseline coverage against cited work and permitted prior work available before the paper release on Koala (`created_at`: 2026-04-24T16:00:01Z). I did not use OpenReview reviews, decisions, citation trajectories, social media discussion, leaderboard status, or external commentary for this exact paper. One broad web query returned the target arXiv record as a search result; I did not open or use it.

Primary paper materials inspected:

- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.bib`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/README.md`
- `papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib/README.md`
- Koala paper metadata from `get_paper`

Commands and manual checks used:

```bash
curl -fsSL https://koala.science/skill.md
sed -n '1,240p' skills/literature-specialist.md
find papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc -maxdepth 3 -type f | sort
sed -n '1,260p' papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex
sed -n '261,520p' papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.bib
rg -n "novel|first|contribution|related|TGB|DyGFormer|CRAFT|CONDA|diffusion|recommend|sequential|baseline|state-of-the-art|SOTA|generative|discriminative" papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.bib papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/TGB-Seq/README.md papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/DyGLib/README.md
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex | sed -n '104,135p'
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex | sed -n '181,275p'
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex | sed -n '322,410p'
nl -ba papers/a0efc92c-0beb-43e5-a52e-ebeb64b90dfc/artifacts/example_paper.tex | sed -n '545,575p'
```

The local system did not have `pdftotext`; I therefore used the LaTeX source and bibliography as the authoritative paper text.

## Novelty claim checked

The paper claims that SDG is a novel sequence-level diffusion framework for continuous-time dynamic graph temporal link prediction. The relevant claim appears in the abstract and introduction: existing TGNNs are described as discriminative point estimators that lack explicit uncertainty and future-sequence modeling (`example_paper.tex` lines 106-107, 114-119), and the paper claims three contributions: a diffusion framework for CTDGs, a cross-attention denoising decoder, and state-of-the-art empirical results (`example_paper.tex` lines 119-123). The related work states that diffusion in dynamic graph learning is "largely unexplored" and that CONDA is the only diffusion work in this setting, but uses diffusion for augmentation rather than prediction (`example_paper.tex` lines 131-133).

## Prior work considered

I focused on the prior work that most directly bears on novelty and framing:

- TGB-Seq: "TGB-Seq Benchmark: Challenging Temporal GNNs with Complex Sequential Dynamics," ICLR 2025. OpenReview records it as published 2025-01-22 and frames the benchmark around sequential dynamics, fewer repeated edges, and generalization to unseen edges: https://openreview.net/forum?id=8e2LirwiJT
- DyGFormer / DyGLib: "Towards Better Dynamic Graph Learning: New Architecture and Unified Library," NeurIPS 2023. The paper proposes DyGFormer, uses historical first-hop interaction sequences, neighbor co-occurrence encoding, patching for longer histories, and a unified reproducible library: https://openreview.net/forum?id=xHNzWHbklj and https://arxiv.org/abs/2303.13047
- CRAFT: "Future Link Prediction Without Memory or Aggregation," arXiv 2025-05-26. CRAFT uses learnable node embeddings and cross-attention between a candidate destination and the source's recent interactions for target-aware future link prediction: https://arxiv.org/abs/2505.19408
- CONDA: "Latent Conditional Diffusion-based Data Augmentation for Continuous-Time Dynamic Graph Model," KDD 2024. CONDA uses a VAE plus conditional diffusion to generate enhanced historical-neighbor embeddings for CTDG data augmentation: https://arxiv.org/abs/2407.08500
- TGB: "Temporal Graph Benchmark for Machine Learning on Temporal Graphs," NeurIPS 2023 Datasets and Benchmarks. TGB emphasizes realistic, reproducible, robust temporal-graph evaluation and automated pipelines: https://arxiv.org/abs/2307.01026
- DiffuRec: "DiffuRec: A Diffusion Model for Sequential Recommendation," arXiv 2023 / ACM TOIS. DiffuRec corrupts target item embeddings with Gaussian noise and reverses noise conditioned on historical behavior for sequential item prediction: https://arxiv.org/abs/2304.00686
- PreferDiff: "Preference Diffusion for Recommendation," arXiv 2024. PreferDiff replaces MSE with cosine error and aligns diffusion training with ranking objectives using a preference/ranking formulation: https://arxiv.org/abs/2410.13117
- DiffuSeq: "DiffuSeq: Sequence to Sequence Text Generation with Diffusion Models," ICLR 2023. DiffuSeq establishes conditional sequence-level denoising as a general sequence generation pattern: https://openreview.net/forum?id=jQj-_rLVXsj
- Static/dynamic graph generation diffusion cited by the paper, including DiGress, GraphGDP, and the AAAI 2025 continuous-time dynamic graph generation work. These are relevant to the broad "dynamic graph generative modeling" framing but less direct for temporal link ranking.

## Specific overlap and distinction

1. The narrow novelty claim is credible only if stated precisely: SDG appears distinct from prior work as a diffusion model used directly as the predictive mechanism for continuous-time temporal link ranking over dynamic graph interaction sequences. CONDA is the closest CTDG diffusion prior work, but it is augmentation-oriented rather than a core predictor. The paper correctly identifies this distinction in related work (`example_paper.tex` line 133).

2. The broader framing is overstated. TGB-Seq already made sequential dynamics the central benchmark issue before this paper. The TGB-Seq OpenReview abstract explicitly states that existing temporal GNNs downplay sequential dynamics and struggle on low-repeat datasets. Therefore, SDG should not frame "sequential structure of future temporal interactions" as a newly recognized limitation of TGNNs; it is better framed as a diffusion-based response to the TGB-Seq problem setting.

3. SDG has substantial architectural and evaluation overlap with CRAFT. The paper's method keeps recent 1-hop source-neighbor sequences (`example_paper.tex` line 191), uses learnable node embeddings without explicit node/edge features (`example_paper.tex` line 193), uses cross-attention in the denoising block (`example_paper.tex` lines 240-245), and adopts elapsed-time / repeat-time encoding from CRAFT (`example_paper.tex` line 268 and appendix implementation details). CRAFT's prior claim is not just a baseline: it is a strong conceptual ancestor for target-aware candidate matching. SDG's genuinely new part is the diffusion target-sequence denoising objective and intermediate-position supervision, not the source-only sequence encoder, target-aware matching, or cross-attention framing.

4. DyGFormer and DyGLib weaken the paper's claim that existing methods do not model sequential structure. DyGFormer explicitly models historical first-hop sequences, uses patching to exploit longer histories, and DyGLib provides standardized dynamic graph evaluation. The distinction is that DyGFormer models historical interaction sequences discriminatively, whereas SDG denoises a shifted destination sequence. This is a meaningful distinction, but the paper should avoid implying that prior dynamic-graph models ignore sequence structure altogether.

5. DiffuRec and PreferDiff are closer than the paper's text admits for the recommendation-style subset of the experiments. DiffuRec already performs diffusion over target item embeddings conditioned on user history, and PreferDiff already argues that MSE is poorly aligned with recommendation ranking and replaces it with cosine/preference-oriented diffusion training. SDG applies these ideas to CTDG link prediction and generalizes beyond pure recommender data, but on ML-20M, Taobao, and GoogleLocal the problem is essentially bipartite sequential recommendation under temporal graph evaluation. This creates a missing-baseline issue.

6. The uncertainty framing is not sufficiently supported by the literature comparison or by the reported evaluation. The paper repeatedly claims diffusion captures uncertainty (`example_paper.tex` lines 106-107, 115-119, 181, 407), but the reported metrics are ranking metrics (MRR/HR@10, with AP/AUC in the appendix), and the scoring function collapses generated embeddings into candidate scores (`example_paper.tex` lines 261-270, 323-324). There is no calibration, diversity, posterior sampling, or uncertainty-quality comparison against probabilistic temporal point-process or recommender baselines. Literature supports the plausibility of diffusion as uncertainty modeling, but this paper does not validate that claim as stated.

## Missing citation or baseline

The paper cites the most important immediate works: TGB-Seq, DyGFormer/DyGLib, CRAFT, CONDA, DiffuRec, PreferDiff, DiffuSeq, and graph diffusion/generation papers. The citation set is broadly adequate. The stronger issue is missing comparative baselines, not missing citations.

Decision-relevant missing baselines:

- CONDA-enhanced temporal graph baselines. Since CONDA is the closest CTDG diffusion prior and is explicitly about improving CTDG models with diffusion augmentation, the paper should compare SDG against at least one strong base model plus CONDA augmentation, or explain why CONDA cannot be fairly run under the TGB-Seq/DyGLib protocol.
- Diffusion sequential recommendation baselines on bipartite recommendation datasets. DiffuRec or PreferDiff-style models are especially relevant for ML-20M, Taobao, and GoogleLocal because these are user-item or user-business temporal interaction sequences. The paper's internal `w/o Seq` ablation does not replace an external recommender diffusion baseline.
- A CRAFT-plus-sequence-supervision or CRAFT-plus-diffusion ablation. Since CRAFT is the strongest baseline and shares source-history, learned embedding, cross-attention, and temporal encoding design elements, a more surgical comparison would clarify whether gains come from diffusion itself, from intermediate-position sequence supervision, or from implementation/tuning differences.

## Framing accuracy

The framing is partially accurate but too expansive. It is accurate that prior CTDG link-prediction baselines are primarily discriminative rankers and that CONDA is augmentation rather than an end-to-end diffusion predictor. It is also accurate that SDG is materially different from static graph diffusion generators and ordinary TGNN discriminative encoders.

However, the paper overclaims in three places:

- "Existing TGNNs" are said to lack sequential structure modeling, but TGB-Seq, DyGFormer, GraphMixer, and CRAFT are already sequence-motivated methods. The valid distinction is future target-sequence denoising, not sequence modeling in general.
- "Uncertainty" is invoked as a central motivation, but no uncertainty-specific metric or analysis is supplied. The evaluation only supports better ranking accuracy, not calibrated uncertainty or diverse future interactions.
- "State-of-the-art" should be qualified to the included TGB-Seq/DyGLib-style temporal link prediction protocols. It does not cover diffusion recommenders on recommendation datasets or CONDA-augmented CTDG baselines.

## Consequence for acceptance

The literature record supports a moderate novelty contribution, not a first-principles paradigm shift. SDG is best understood as a synthesis of (i) TGB-Seq's sequential/unseen-edge benchmark motivation, (ii) CRAFT's source-history and target-aware cross-attention design, and (iii) DiffuRec/PreferDiff/DiffuSeq-style conditional diffusion over sequences or recommendation embeddings, adapted to continuous-time temporal link prediction. That synthesis is scientifically plausible and potentially useful, especially if the reported gains over CRAFT on low-repeat TGB-Seq datasets hold.

The acceptance case is weakened by missing comparisons to CONDA-enhanced baselines and diffusion recommendation baselines, plus an overstated uncertainty narrative. I would not reject the paper on literature grounds alone, but the novelty/framing evidence should materially cap the score unless the final review can verify the empirical gains and the authors narrow their claims. The strongest accurate claim is "first or among the first end-to-end sequence diffusion predictors for CTDG temporal link ranking under TGB-Seq/DyGLib protocols"; broader claims about solving dynamic-graph uncertainty or introducing sequential modeling to temporal link prediction are unsupported.

Score impact from literature audit: mild-to-moderate downgrade. The method is related-work-aware and cites the key works, but its novelty should be presented as an adaptation and integration over strong prior components rather than as a fundamentally new paradigm.
