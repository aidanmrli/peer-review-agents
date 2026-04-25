# Implementation Auditor Report

Paper: `75c4a4bd-208f-451a-8ed8-121748a738c7`
Title: "Plain Transformers are Surprisingly Powerful Link Predictors"
Role: Implementation Auditor

## Bottom line

The submitted Koala/arXiv artifact does not contain an executable implementation of PENCIL. The platform metadata reports no code repository, the source tarball contains only LaTeX sources, style files, bibliography, and figures, and the paper text provides no author code URL. As a result, the central empirical claims about PENCIL's reported link-prediction performance, efficiency, stability, and ablations are not independently reproducible from the provided artifacts.

Severity for acceptance: high to critical. The paper's main case is empirical and implementation-dependent, but reviewers cannot run training, evaluation, preprocessing, negative sampling, HeaRT evaluation, batching benchmarks, ablations, or table generation from the submitted materials.

## Claim being audited

The implementation audit focuses on the paper's core implementation-dependent claims:

- PENCIL is an encoder-only plain Transformer link predictor operating on sampled local subgraphs without handcrafted heuristics or node IDs.
- PENCIL achieves state-of-the-art or competitive results on Planetoid, OGB link prediction, and HeaRT settings.
- PENCIL is parameter-efficient and training-efficient, including claims such as strong `ogbl-ppa` performance and fast convergence on large datasets.
- PENCIL's pairwise heuristic estimation, multiplicative residual, initialization, batching-time, and per-batch runtime analyses support the mechanism and scalability claims.

## Artifact inventory

Local artifact directory inspected:

`papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/`

Files present:

- `00README.json`
- `paper.pdf`
- `source.tar.gz`
- `main.tex`
- `main.bib`
- `icml2026.bst`
- `icml2026.sty`
- `algorithm.sty`
- `algorithmic.sty`
- `fancyhdr.sty`
- `figures/PENCIL.png`
- `figures/adjacency_row.png`
- `figures/averaged_test_mrr.png`
- `figures/batching_time_and_memory_comparison.png`
- `figures/citeseer_heuristic_rmse.png`
- `figures/combined_layers_vs_hk.png`
- `figures/cora_heuristic_rmse.png`
- `figures/ogbl_ppa_comparison.png`

Artifact type counts from `find ... -printf '%f\n' | awk ...`: 1 `.bib`, 1 `.bst`, 1 `.gz`, 1 `.json`, 1 `.pdf`, 8 `.png`, 4 `.sty`, 1 `.tex`.

The tarball listing has 17 entries and matches the LaTeX/figure bundle:

```bash
tar -tzf papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz | sort
```

Result: only `00README.json`, LaTeX/style/bibliography files, and `figures/*.png`.

Executable implementation artifacts absent:

- no `.py`, `.ipynb`, `.sh`, `.yaml`, `.yml`, `.toml`, `.cfg`, `requirements.txt`, `environment.yml`, `Dockerfile`, `Makefile`, checkpoints, cached data, evaluator scripts, preprocessing scripts, or table-generation scripts;
- no repository checkout or source package for PENCIL;
- no runnable command line entrypoint.

## Platform metadata

MCP command:

```text
mcp__koala__.get_paper({"paper_id":"75c4a4bd-208f-451a-8ed8-121748a738c7"})
```

Relevant metadata:

- `title`: "Plain Transformers are Surprisingly Powerful Link Predictors"
- `arxiv_id`: `2602.01553`
- `status`: `in_review`
- `pdf_url`: `/storage/pdfs/75c4a4bd-208f-451a-8ed8-121748a738c7.pdf`
- `tarball_url`: `/storage/tarballs/75c4a4bd-208f-451a-8ed8-121748a738c7.tar.gz`
- `github_repo_url`: `null`
- `github_urls`: `[]`

This confirms that the platform does not associate any GitHub implementation with the paper.

## Source and link inspection

Commands used:

```bash
rg -n "github|gitlab|bitbucket|code|repository|repo|artifact|implementation|available|supplement|supplementary|anonymous|http|https|doi|zenodo|huggingface|drive|dropbox" \
  papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex \
  papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.bib

rg -n -F "\\url" papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
rg -n -F "\\href" papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
rg -n -i "github|gitlab|bitbucket|code is available|source code|anonymous|artifact|repository" \
  papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
```

Findings:

- `main.tex` contains anonymous author metadata only at lines 101, 103, and 106; no code URL, repository URL, or supplementary artifact URL appears in the paper source.
- GitHub URLs found in `main.bib` are in abstracts/metadata for prior work, not for PENCIL:
  - `https://github.com/GraphPKU/Refined-GAE` appears in the Refined-GAE bibliography entry at `main.bib:594`.
  - `https://github.com/devnkong/GOAT` appears in the GOAT bibliography entry at `main.bib:800`.
  - These are baseline/prior-work repositories, not the submitted PENCIL implementation.
- The GraphGPT citation used for the ShaDowKHop sampler points to OpenReview, not an implementation repository, at `main.bib:361-368`.

I queried the GitHub API for the two GitHub repositories linked in the bibliography without cloning or writing files:

```bash
curl -fsSL https://api.github.com/repos/GraphPKU/Refined-GAE | sed -n '1,80p'
curl -fsSL https://api.github.com/repos/devnkong/GOAT | sed -n '1,80p'
```

The API metadata identifies them as `GraphPKU/Refined-GAE` ("Reconsidering the Performance of GAE in Link Prediction") and `devnkong/GOAT` ("Official implementation of GOAT model (ICML2023)"). They do not implement PENCIL and cannot verify the paper's method-code consistency.

## Paper-to-code match

No paper-to-code match can be established, because no PENCIL code is provided.

The paper does describe implementation-level choices in `main.tex`:

- Graph encoding uses endpoint-fixed, randomly indexed sampled subgraphs with one-hot IDs, adjacency rows, and role flags (`main.tex:190-205`).
- The model reconstructs adjacency from the token encoding and adds a multiplicative residual branch (`main.tex:207-220`).
- Practical implementation allegedly uses the ShaDowKHop sampler from GraphGPT, Hugging Face BERT layers, PyTorch, and PyG (`main.tex:776`).
- The main experiments are repeated with five random seeds on Planetoid and three random seeds on OGB (`main.tex:372`).
- The paper provides hyperparameter tables for pairwise heuristic estimation (`main.tex:791-808`), original benchmark settings (`main.tex:853-870`), and HeaRT settings (`main.tex:875-894`).

These descriptions are not backed by executable source files. I therefore could not verify:

- whether the submitted model actually uses a standard BERT encoder as claimed;
- whether the adjacency reconstruction matches the mathematical description;
- whether the multiplicative residual branch is implemented as described;
- whether the input projection matrix is orthogonally initialized and frozen;
- whether optional node features are incorporated correctly;
- whether training/evaluation pipelines reproduce the reported tables;
- whether parameter counts and timing measurements are computed from the stated models.

## Reproducibility details present in the paper

The paper includes partial textual details:

- Dataset/split protocol: Planetoid fixed split from Li et al. 2023 and OGB official splits (`main.tex:414`).
- HeaRT protocol description: same training set, different curated negatives for evaluation (`main.tex:414`).
- Random seed count: five for Planetoid, three for OGB (`main.tex:372`).
- PENCIL hyperparameters:
  - original benchmark settings: sampling configuration, hidden size, intermediate size, layers, attention heads, effective batch size, learning rate, weight decay, epochs (`main.tex:859-868`);
  - HeaRT settings: analogous table including `ogbl-ddi` (`main.tex:882-891`);
  - pairwise heuristic estimation: sampling, hidden size, intermediate size, attention heads, effective batch size, learning rate, weight decay (`main.tex:797-806`).
- Heuristic target preprocessing: log transforms, normalization/standardization, percentile clipping, Katz beta `0.005`, PageRank alpha `0.85`, SPD disconnected-pair treatment (`main.tex:789-845`).
- Hardware class: A5000/A6000 for Planetoid and A100 for OGB, parallelized across 4/8 GPUs (`main.tex:849`).

These are useful descriptions but are not sufficient for reproduction without code.

## Reproducibility blockers

Critical blockers:

1. No executable PENCIL implementation is provided.
   - There is no model code, sampler wrapper, training loop, evaluation loop, table generation, ablation code, or benchmark script.

2. No environment specification is provided.
   - No pinned versions for Python, PyTorch, PyG, Hugging Face Transformers, OGB, CUDA, GraphGPT sampler dependency, or HeaRT evaluation dependency.
   - Local dependency probe showed `torch`, `torch_geometric`, `transformers`, `ogb`, and `networkx` are not installed in this reviewer environment, but this is secondary; the primary failure is absence of code/environment files.

3. No exact random seeds are provided.
   - The paper reports seed counts, but not seed values.
   - This matters because the method uses stochastic subgraph sampling and randomized endpoint-constrained node indexing (`main.tex:191`, `main.tex:247-260`).

4. Dataset preprocessing is underspecified operationally.
   - The paper cites fixed/official splits and HeaRT negatives, but does not provide scripts to download, preprocess, cache, or validate the exact data objects.
   - The pairwise heuristic regression pipeline involves full-graph target computation and percentile clipping; without scripts, percentile computation scope and train/validation/test leakage handling cannot be audited.

5. Evaluator scripts are absent.
   - No OGB evaluator invocation, HeaRT evaluator invocation, MRR/Hits calculation code, negative sampling code, or per-dataset metric aggregation scripts are provided.

6. Hyperparameter tables are incomplete as execution specifications.
   - Missing optimizer name, scheduler, warmup, gradient accumulation details behind "effective batch size", dropout, attention implementation, mixed precision, early stopping/checkpoint selection, validation cadence, dataloader worker settings, and exact negative sampling for training.
   - The paper says for `ogbl-ppa` HeaRT it used a single negative link per positive link and the optimal checkpoint from the original benchmark setting (`main.tex:849`), but the checkpoint and selection procedure are not supplied.

7. Table and figure generation is not reproducible.
   - The artifact includes rendered PNGs and LaTeX tables but no raw result logs, CSVs, notebooks, or plotting/table scripts.
   - The parameter-efficiency, convergence, initialization ablation, multiplicative residual ablation, batching-time, and training/inference-time results cannot be regenerated.

8. Baseline comparability cannot be audited.
   - The paper mixes baseline results from Li et al. 2023 and results quoted from LPFormer, MPLP+, and Refined-GAE (`main.tex:414`).
   - No scripts reproduce the baseline extraction, metric normalization, or categorization logic used in the tables.

## Paper-to-code discrepancies

Because no implementation is released, direct paper-code discrepancies cannot be tested. The audit still identifies documentation-to-reproducibility gaps:

- The paper states "All codes are written in Pytorch and Pytorch Geometric" (`main.tex:776`), but no such code is present in the artifact.
- The paper relies on a GraphGPT ShaDowKHop sampler implementation (`main.tex:776`), but no dependency pointer, version pin, copied code, or integration wrapper is included.
- The paper reports standard deviations across repeated runs (`main.tex:372`, tables), but does not provide run logs or exact seeds.
- The paper claims hardware-efficient batching and timing behavior (`main.tex:993-1020`) but does not provide benchmarking scripts or raw measurements.
- The paper says PENCIL does not need costly offline PE/SE computation (`main.tex:151`), but the artifact does not let reviewers inspect whether any hidden preprocessing or caching is used in the actual pipeline.

## What cannot be independently verified

The following claims cannot be independently verified from the submitted artifact:

- PENCIL's reported `cora`, `citeseer`, `pubmed`, `ogbl-collab`, `ogbl-ppa`, `ogbl-citation2`, and `ogbl-ddi` results.
- The claim that PENCIL achieves state-of-the-art results on `cora` and `ogbl-ppa` in the original setting and on `ogbl-ppa`/`ogbl-ddi` under HeaRT.
- The stability claim, especially the low `ogbl-ppa` variance.
- The fast-convergence claim for `ogbl-citation2`, `ogbl-ddi`, and `ogbl-ppa`.
- The pairwise heuristic estimation results for CN, AA, RA, Katz, SPD, and PageRank.
- The multiplicative residual ablation.
- The input projection initialization ablation.
- The batching time/memory comparison and training/inference time table.
- Any paper-code consistency claim about the model architecture.

## Commands run

Representative commands used for this audit:

```bash
sed -n '1,240p' skills/implementation-auditor.md
find papers/75c4a4bd-208f-451a-8ed8-121748a738c7 -maxdepth 3 -type f -print | sort
sed -n '1,220p' papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/00README.json
tar -tzf papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz | sort | sed -n '1,240p'
tar -tzvf papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz | sed -n '1,120p'
find papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts -type f -printf '%p\t%s bytes\n' | sort
rg -n "seed|random|epoch|learning rate|optimizer|batch|hyperparameter|ShaDow|Hugging|PyTorch|Pytorch|PyG|evaluator|HeaRT|OGB|negative|preprocess|split|GPU|hardware|memory|parameter" papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '180,230p'
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '760,940p'
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '980,1045p'
rg -n -i "github|gitlab|bitbucket|code is available|source code|anonymous|artifact|repository" papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
rg -n "github\.com|gitlab\.com|bitbucket\.org|huggingface\.co|zenodo\.org|osf\.io|drive\.google|dropbox\.com|anonymous\.4open|openreview\.net/attachment|raw\.githubusercontent" papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.bib
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
curl -fsSL https://api.github.com/repos/GraphPKU/Refined-GAE | sed -n '1,80p'
curl -fsSL https://api.github.com/repos/devnkong/GOAT | sed -n '1,80p'
```

Koala MCP command used:

```text
get_paper("75c4a4bd-208f-451a-8ed8-121748a738c7")
```

## Final assessment

The implementation package is not reproducible. It is a paper-source package, not a code artifact. The textual appendix is materially better than nothing because it reports major hyperparameters and some preprocessing choices, but it does not cross the threshold for independent reproduction of an implementation-heavy empirical paper. The absence of an executable PENCIL implementation, environment specification, exact seeds, data preprocessing/evaluation scripts, and result-generation scripts should materially reduce confidence in the reported empirical claims.
