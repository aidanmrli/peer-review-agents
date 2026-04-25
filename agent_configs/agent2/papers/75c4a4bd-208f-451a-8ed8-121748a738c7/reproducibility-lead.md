# Reproducibility Lead Report

Paper: `75c4a4bd-208f-451a-8ed8-121748a738c7`  
Title: "Plain Transformers are Surprisingly Powerful Link Predictors"  
Role: Reproducibility Lead  

## Claim Being Tested

The paper claims that PENCIL is a plain BERT-style Transformer link predictor over sampled local subgraphs that avoids hand-crafted heuristics and persistent node IDs, while achieving state-of-the-art or competitive performance with strong parameter/training efficiency and a theoretical unification of structural link-prediction heuristics.

## Team Protocol

I coordinated five independent role checks before preparing any public comment:

- Independent Reproducer A checked whether the central Tables 1/2 claims can be rerun from artifacts.
- Independent Reproducer B independently recomputed table rankings, standard-deviation claims, ablation arithmetic, and parameter/time ratios.
- Implementation Auditor inspected code availability, artifact contents, dependency information, evaluator availability, preprocessing, logs, and code-paper traceability.
- Correctness Specialist checked definitions, formulas, theorem/proof logic, metrics, evaluation protocols, and table-derived conclusions.
- Literature Specialist checked novelty and framing against the cited prior work, without using leaked future information.

Reports written:

- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/independent-reproducer-a.md`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/independent-reproducer-b.md`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/implementation-auditor.md`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/correctness-specialist.md`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/literature-specialist.md`

## Artifacts Inspected

Local artifact directory:

- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/paper.pdf`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex`
- `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.bib`
- static figure files under `papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/figures/`

Koala metadata:

- `github_repo_url: null`
- `github_urls: []`
- `status: in_review`

The source tarball contains LaTeX, style files, bibliography, and rendered figures only. No runnable implementation, configs, dependency files, evaluator scripts, raw logs, checkpoints, or table-generation scripts are present.

## Commands and Environment

Representative commands:

```bash
curl -fsSL https://koala.science/skill.md
rg --files papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts
tar -tzf papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/source.tar.gz
rg -n "github|repository|code|ShaDow|Pytorch|PyG|seed|random|HeaRT|negative|checkpoint" papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '329,420p'
nl -ba papers/75c4a4bd-208f-451a-8ed8-121748a738c7/artifacts/main.tex | sed -n '613,666p'
```

Environment observations:

- Working directory: `/home/mila/l/lia/peer-review-agents/agent_configs/agent2`
- Python: `3.12.12`
- `pdftotext` and `pdfinfo` were not installed, so the LaTeX source was the primary paper text.
- No code environment could be built because there is no submitted code.

## Independent Reproduction Outcomes

Independent Reproducer A outcome: blocked / weak reproducibility.

This role found no runnable PENCIL implementation, exact seed list, executable config, evaluator invocation, dataset preprocessing, raw logs, checkpoint, or table-generation script. The empirical Tables 1/2 claims could not be rerun.

Independent Reproducer B outcome: blocked for full empirical reproduction; partial static table verification; several broad claims contradicted.

This role independently recomputed table ranks and found that, counting the best PENCIL-family row, PENCIL wins 2 of 6 original-setting columns and 2 of 7 HeaRT columns. The multiplicative-residual ablation arithmetic is correct, but claims of broad outperformance and consistently lower standard deviations are not supported.

The two independent reproducer roles agree that the central empirical claim is weakly reproducible from artifacts only. Neither role reproduced a training or evaluation result.

## Implementation Findings

The implementation audit found:

- No executable implementation or official GitHub repository is provided.
- The artifact package contains only paper source, style files, bibliography, and static figures.
- The paper mentions ShaDowKHop, GraphGPT, Hugging Face BERT, PyTorch, and PyG, but provides no versions, dependency files, commits, scripts, or command lines.
- Exact seed values, optimizer details, scheduler, dropout, gradient accumulation, validation cadence, checkpoint selection, and evaluator invocations are missing.
- Baseline extraction and table-generation procedures are not auditable.

This is a high-to-critical reproducibility failure because the paper's main acceptance case is empirical and implementation-dependent.

## Correctness Findings

The correctness specialist identified major issues:

- The NBFNet degeneration proof is algebraically wrong. The paper sets `T_k` to zero, but Eq. (2) defines `Z^(k) = T_k(H^(k-1))` and `H^(k) = Z^(k) + P_k(A Z^(k))`. If `T_k=0`, the propagation branch receives `A 0`, not `H^(k-1)`, so it does not reduce to message passing.
- The local-heuristic estimator proof repeats the same zero-propagation error.
- The global-heuristic corollary imports full-graph NBFNet/Bellman-Ford reasoning into a fixed-budget sampled-subgraph model without proving that PENCIL has the required graph support, semiring operators, or iterations.
- The `ogbl-ppa` HeaRT entry is nonstandard: the appendix says it uses one negative link per positive and reuses the original-setting checkpoint because HeaRT curated negatives are computationally prohibitive.
- The "plain Transformer" framing is overstated because every layer includes an explicit adjacency propagation residual, and the appendix ablation says an explicit structural prior is necessary for every layer.
- The main text and appendix are inconsistent about the residual adjacency operator: the main text discusses raw adjacency for notation, while the appendix adds identity links, task-token zero columns, and row normalization in the implemented operator.

## Table and Metric Checks

Reproducer B recomputed the main table ranks:

- Original setting: PENCIL-family best row is first on `cora` and `ogbl-ppa`, but trails on `citeseer`, `pubmed`, `ogbl-collab`, and `ogbl-citation2`.
- HeaRT setting: PENCIL-family best row is first on `ogbl-ppa` and `ogbl-ddi`, but trails on `cora`, `citeseer`, `pubmed`, `ogbl-collab`, and `ogbl-citation2`.
- Exact gaps from the best non-PENCIL table entry include `citeseer` original `-17.91`, `pubmed` original `-6.39`, `ogbl-citation2` original `-3.86`, `citeseer` HeaRT `-11.85`, and `ogbl-collab` HeaRT `-2.22`.
- The claim of consistently lower standard deviations is contradicted by many columns; it is strongly supported mainly for original-setting `ogbl-ppa`.
- The multiplicative-residual gain row is arithmetically correct.
- Exact `22x` to `146x` parameter savings and `6.7x` to `40x` epoch savings cannot be independently recovered from source data in the artifacts.

## Literature Findings

The literature specialist found a defensible but narrower novelty core:

- PENCIL is plausibly a useful link-centric, sampled-subgraph, endpoint-canonical Transformer scorer that avoids global PE/SE caches and persistent graph-scale ID embeddings.
- The broad "plain Transformer" framing is too strong because the method uses adjacency-row tokenization and a graph propagation residual.
- Node-adjacency tokenization, GraphGPT/ShaDowKHop, LPFormer, MPLP/MPLP+, Refined-GAE, NBFNet, SEAL/LRP, NCN/NCNC, and graph Transformer prior work cover much of the conceptual ground.
- The defensible novelty is not plain Transformers on graphs in general, but a deployment-constrained link-prediction adaptation with sampled local subgraphs and an explicit structural residual.

## Final Synthesis and Score Impact

Reproducibility classification: weak reproducibility.

The empirical results may be correct, but they are not independently reproducible from the submitted artifacts. The only reproducible checks are static source/table checks, and those checks narrow or contradict several broad claims. The proof errors and nonstandard `ogbl-ppa` HeaRT protocol issue are decision-relevant because they affect the theoretical and empirical headline claims.

Score impact: substantial negative. I would not treat the submission as a reproducible strong empirical result unless the authors provide executable PENCIL code, exact seeds/configs, evaluator scripts, logs or checkpoints, corrected HeaRT protocol reporting, and corrected theory for the NBFNet/local-heuristic claims.
