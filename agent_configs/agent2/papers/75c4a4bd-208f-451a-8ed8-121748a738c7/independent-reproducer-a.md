# Independent Reproducer A Report

Paper: 75c4a4bd-208f-451a-8ed8-121748a738c7  
Title: Plain Transformers are Surprisingly Powerful Link Predictors  
Role: Independent Reproducer A  
Report time: 2026-04-24T21:58:01-04:00

## Claim Attempted

I attempted to reproduce the central empirical claim: PENCIL, an encoder-only plain Transformer over sampled link-centric local subgraphs, achieves surprisingly strong link-prediction performance without hand-crafted heuristic features or node IDs. The decision-relevant claims are concentrated in:

- Abstract, `artifacts/main.tex:132`: PENCIL is claimed to outperform heuristic-informed GNNs and be more parameter-efficient than ID-embedding alternatives while remaining competitive across benchmarks.
- Contributions, `artifacts/main.tex:151-153`: PENCIL is claimed to use a standard BERT-style encoder on fixed-budget sampled neighborhoods and to exceed other methods with 22x to 146x fewer learnable parameters.
- Table 1, `artifacts/main.tex:333-365`: original benchmark results. Key PENCIL entries include PENCIL without features at Cora MRR 42.23 +/- 1.98, ogbl-collab Hits@50 66.88 +/- 0.34, ogbl-ppa Hits@100 73.85 +/- 0.40; PENCIL with features at ogbl-ppa Hits@100 79.54 +/- 0.07.
- Table 2, `artifacts/main.tex:375-401`: HeaRT results. Key PENCIL entries include PENCIL without features at ogbl-ppa MRR 44.57 +/- 0.15 and ogbl-ddi MRR 14.07 +/- 0.24; PENCIL with features at ogbl-ppa MRR 45.43 +/- 0.31.
- Main-results text, `artifacts/main.tex:414-418`: benchmark protocols, baseline provenance, claimed state-of-the-art results, convergence claims, and structural-sufficiency interpretation.

## Setup Used

I used only the provided Koala metadata, local paper artifacts, and permitted prior literature references cited by the paper. I did not use OpenReview reviews, later decisions, citation counts, social media, or post-publication commentary.

Local environment observations:

- Working directory: `/home/mila/l/lia/peer-review-agents/agent_configs/agent2`
- OS: `Linux cn-f003.server.mila.quebec 5.15.0-173-generic #183-Ubuntu SMP Fri Mar 6 13:29:34 UTC 2026 x86_64`
- Python: `Python 3.12.12`
- GPU query: `nvidia-smi` was not installed on this node.
- `pdftotext` was not installed, so I used the LaTeX source as the primary paper text.

Artifact checksums:

```text
802fafc8fd8bf3c0c287e9dbeba898add5bac0a29a8016ebcfa9481fb7a2a3d0  artifacts/paper.pdf
7eaf46b3a1ce8bfaaca6802c6263a270c719520a36d02fb064550ecaf1af32cd  artifacts/source.tar.gz
9f7585e3b4c1db20307682758d362d5e0c665d745e5f2a0088cb3b8af5a0b79d  artifacts/main.tex
```

## Commands and Observations

Loaded role and platform instructions:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,220p' skills/independent-reproducer-a.md
```

Inventoried local artifacts:

```bash
rg --files papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts
tar -tzf papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz
```

Observed artifact contents:

- `main.tex`, `main.bib`, ICML style files, `paper.pdf`, `source.tar.gz`, and PNG figures.
- The tarball contains the same paper-source files and figures.
- No training scripts, evaluation scripts, Python modules, notebooks, shell scripts, environment files, config files, checkpoints, raw logs, processed datasets, or metric outputs were present.

Searched for runnable code/config files:

```bash
rg --files papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts | rg '(\.py$|\.ipynb$|\.sh$|requirements|environment|conda|setup|pyproject|\.ya?ml$|config|checkpoint|\.pt$|\.pth$|\.csv$|\.json$|README|readme)'
```

Observed result:

```text
papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/00README.json
```

Checked for repository links and implementation URLs:

```bash
rg -n "https?://|github|GitHub|repository|repo|code|Code|available|artifact" \
  papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex \
  papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.bib
```

Observed result: no PENCIL implementation repository was identified in `main.tex`. GitHub mentions found in `main.bib` were for cited prior work, e.g. Refined-GAE and GOAT, not this paper's PENCIL code.

Checked Koala metadata with `get_paper`:

```text
github_repo_url: null
github_urls: []
status: in_review
arxiv_id: 2602.01553
```

This confirms that Koala did not attach a code repository for this submission.

Checked documented implementation details:

```bash
rg -n "seed|random|optimizer|Adam|scheduler|dropout|warmup|checkpoint|batch|evaluator|negative|split|preprocess|ShaDow|GraphGPT|Hugging Face|PyTorch|Pytorch|PyG|OGB|HeaRT" \
  papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
```

Relevant paper details located:

- `artifacts/main.tex:372`: experiments use five random seeds on Planetoid and three random seeds on OGB datasets, but exact seed values are not provided.
- `artifacts/main.tex:414`: Planetoid uses fixed splits from Li et al. 2023; OGB uses official splits; HeaRT uses curated negatives; baseline values are partly imported from prior papers.
- `artifacts/main.tex:776`: implementation says ShaDowKHop sampler from GraphGPT, Hugging Face BERT, Pytorch, and PyG are used, but no package versions, commits, scripts, or command lines are provided.
- `artifacts/main.tex:789-806`: heuristic-estimation setup gives preprocessing at a prose level and hyperparameters, but no code or exact target-generation script.
- `artifacts/main.tex:849-892`: original and HeaRT PENCIL hyperparameter tables give sampling configurations, hidden size, intermediate size, number of layers, heads, effective batch size, learning rate, weight decay, and number of epochs.

Checked figure artifacts:

```bash
file papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/figures/*.png
```

Observed result: figures are static PNGs only. They preserve plots visually but do not include source data needed to recompute RMSE/MRR/Hits values.

## Exact Config and Seed Availability

The paper provides partial human-readable configurations, but not an exact executable reproduction specification.

Available for Table 1 original setting, from `artifacts/main.tex:855-868`:

- Sampling configurations: Cora `(2,20)`, Citeseer `(2,20)`, Pubmed `(2,16)`, ogbl-collab `(1,75)`, ogbl-ppa `(1,150)`, ogbl-citation2 `(1,40)`.
- Hidden size: 512 except Citeseer 1024.
- Intermediate size: 2048.
- Layers: Cora 8, Citeseer 2, Pubmed 8, ogbl-collab 8, ogbl-ppa 4, ogbl-citation2 3.
- Heads: 8.
- Effective batch size: 2048, 2048, 4096, 8192, 16384, 65536 respectively.
- Learning rate: 1e-4.
- Weight decay: 0.01.
- Epochs: 150, 150, 80, 10, 15, 0.5 respectively.

Available for Table 2 HeaRT setting, from `artifacts/main.tex:877-891`:

- Sampling configurations: Cora `(2,20)`, Citeseer `(2,20)`, Pubmed `(2,20)`, ogbl-collab `(1,75)`, ogbl-ppa `(1,150)`, ogbl-citation2 `(1,40)`, ogbl-ddi `(1,350)`.
- Hidden size: 512 except Citeseer 1024.
- Intermediate size: 2048.
- Layers: 4, 2, 4, 8, 4, 3, 8 respectively.
- Heads: 8.
- Effective batch size: 2048, 2048, 2048, 8192, 16384, 65536, 4096 respectively.
- Learning rate: 1e-4.
- Weight decay: 0.01.
- Epochs: 150, 300, 250, 10, 15, 0.5, 8 respectively.

Missing for exact reproduction:

- No exact random seed values, despite reporting seed counts.
- No training/evaluation command lines.
- No source implementation of PENCIL, sampler wrapping, batching, loss, evaluator integration, feature/no-feature toggles, or metric aggregation.
- No GraphGPT sampler commit/version or Hugging Face/PyG/Pytorch versions.
- No optimizer identity beyond learning rate and weight decay. The paper does not specify Adam/AdamW betas, epsilon, scheduler, warmup, gradient accumulation, clipping, dropout, precision, or early-stopping/checkpoint-selection rules.
- No processed datasets or HeaRT negative files. The text references Li et al. 2023 and OGB splits, but does not package the splits or evaluator inputs.
- No raw logs, per-seed outputs, checkpoints, or scripts to verify the reported means and standard deviations.
- `artifacts/main.tex:849` states that ogbl-ppa HeaRT used an optimal checkpoint from the original benchmark setting and a single negative link per positive link, but the checkpoint and exact selection rule are absent.

## Reproduction Attempt and Outcome

I could not run a PENCIL training or evaluation job because no runnable implementation was provided in the official artifacts and Koala metadata lists no GitHub repository. There was no documented command to invoke, no dependency file to install from, and no checkpoint or output file against which to compare.

Smallest meaningful unit attempted:

- I verified that the paper source contains the Table 1 and Table 2 metric claims and repeats the complete tables in the appendix (`artifacts/main.tex:896-965`).
- I verified that the paper describes the broad benchmark/evaluator protocol: Planetoid fixed splits from Li et al. 2023, OGB official splits from Hu et al. 2020, HeaRT negatives from Li et al. 2023, and dataset-appropriate MRR/Hits metrics (`artifacts/main.tex:414`).
- I verified that the paper gives partial PENCIL hyperparameters and states the sampler/model libraries (`artifacts/main.tex:776`, `artifacts/main.tex:849-892`).

This is not an empirical reproduction. It is only a paper-source consistency check. It does not recover any Table 1 or Table 2 result from code, data, logs, or checkpoints.

Reproduction outcome: **blocked / weak reproducibility**.

Concrete reason for failure: the central empirical claim depends on a custom PENCIL implementation, sampler integration, datasets/evaluators, per-seed execution, checkpointing, and metric aggregation that are not present in the artifacts and are not linked through Koala metadata.

## Prior Literature Used

Only paper-cited, non-leaking prior references were consulted through `main.bib`:

- Li et al. 2023, "Evaluating Graph Neural Networks for Link Prediction: Current Pitfalls and New Benchmarking" (`li_evaluating_2023`), used by the paper for Planetoid fixed splits, HeaRT evaluation, and some baseline results.
- Hu et al. 2020, "Open Graph Benchmark: Datasets for Machine Learning on Graphs" (`hu2020ogb`), used for OGB splits and metrics.
- Zeng et al. 2021, "Decoupling the Depth and Scope of Graph Neural Networks" (`zeng2021decoupling`), cited for ShaDowKHop.
- Wolf et al. 2020, "Transformers: State-of-the-Art Natural Language Processing" (`wolf-etal-2020-transformers`), cited for Hugging Face Transformers.
- Shomer et al. 2024, "LPFormer: An Adaptive Graph Transformer for Link Prediction" (`shomer_lpformer_2024`), Dong et al. 2024, "Pure Message Passing Can Estimate Common Neighbor for Link Prediction" (`dong_pure_2024`), and Ma et al. 2025, "Reconsidering the Performance of GAE in Link Prediction" (`ma_reconsidering_2025`), used only to identify the paper's stated baseline provenance.

I did not independently fetch or use later venue outcomes, public reviews, citations, or any future-impact signals.

## Final Synthesis and Score Impact

The main empirical claim is not independently reproducible from the provided artifacts. The paper contains clear narrative claims, static tables, static figures, and partial hyperparameter tables, but it does not provide the runnable machinery needed to regenerate Table 1 or Table 2. The absence of a GitHub repository, implementation files, exact seeds, environment specification, dataset/evaluator artifacts, raw logs, and checkpoints is decisive for this role.

I would mark the empirical reproducibility of the central Tables 1/2 claims as weak. The results may be correct, but the artifacts do not let an independent reviewer verify them. This should materially reduce confidence in the paper's acceptance case, especially because the contribution rests heavily on reported empirical superiority and stability rather than on a reproducible released implementation.
