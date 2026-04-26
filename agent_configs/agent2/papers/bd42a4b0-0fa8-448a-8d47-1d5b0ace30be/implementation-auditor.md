# Implementation Auditor Report

Paper: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Role: Implementation Auditor
Audit date: 2026-04-25 America/Toronto

## Bottom Line

The implementation support for the central empirical claims is weak. Koala metadata provides no GitHub repository, the manuscript claims code is publicly available at an anonymous 4open link, but that link is not accessible from this review environment. The official Koala source artifact contains only LaTeX manuscript sources, bibliography/style files, a PDF, and one image. I found no runnable implementation, data preprocessing scripts, training/evaluation entry points, configuration files, seeds, dependency lockfile, or metric code supporting the reported accuracy, sparsity, explanation-size, baseline, ablation, or runtime results.

Severity for acceptance: high to critical. The paper's main contribution is empirical and algorithmic, and its reported results cannot be independently audited against code or scripts from the official artifacts.

## Artifact Inventory

Official Koala metadata inspected with `get_paper`:

- `pdf_url`: `/storage/pdfs/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be.pdf`
- `tarball_url`: `/storage/tarballs/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be.tar.gz`
- `github_repo_url`: `null`
- `github_urls`: `[]`
- `arxiv_id`: `2602.04360`
- `status`: `in_review`

Official source tarball contents, from `curl ...source.tar.gz | tar -tzf -` and the local extracted copy under `papers/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be/artifacts/`:

- Manuscript and metadata: `main.tex`, `00README.json`, `Introduction.tex`, `RelatedWorks.tex`, `Background.tex`, `ProblemFormulation.tex`, `ExperimentalSetup.tex`, `Results.tex`, `Limitations.tex`, `Conclusions.tex`, `macros.tex`, `bibliography.bib`
- Style files: `algorithm.sty`, `algorithmic.sty`, `fancyhdr.sty`, `icml2026.sty`, `icml2026.bst`
- Figure: `imgs/openalex_counts_by_year.png`
- Locally present artifacts: `paper.pdf`, `source.tar.gz`

No implementation files are present in the official source artifact: no `.py`, notebooks, shell scripts, environment files, configs, datasets, trained checkpoints, raw logs, result CSVs, or evaluation outputs.

Declared code artifact in the manuscript:

- `ExperimentalSetup.tex:4-6` states that the implementation is based on PyTorch Geometric, includes dense and sparse variants, and that code is publicly available at `https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716`.
- Access checks:
  - `curl -sS -o /dev/null -w '%{http_code} %{redirect_url}\n' https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716` returned `302 https://anonymous.4open.science/api/repo/CF-HyperGNNExplainer-0716/file/`.
  - `curl -sS -o /dev/null -w '%{http_code}\n' https://anonymous.4open.science/api/repo/CF-HyperGNNExplainer-0716/file/` returned `401`.
  - `curl -fsSL https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716` failed with HTTP 401.
  - `git ls-remote https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716` failed with `fatal: repository ... not found`.

I therefore could not inspect any official code implementation.

## Code Paths Inspected

No code paths could be inspected because no runnable code tree was available. The inspected artifact paths were manuscript paths:

- `ExperimentalSetup.tex` for claimed framework, hardware, datasets, model/training hyperparameters, baseline setup, metrics, and hyperparameter search.
- `ProblemFormulation.tex` for the described perturbation operators, loss, continuous relaxation, thresholding, and V1/V3 variants.
- `Results.tex` for reported accuracy, sparsity, explanation-size, ablation, baseline comparison, and runtime tables.
- `Limitations.tex` for the authors' own discussion of random baseline and sparse-hypergraph weaknesses.
- `00README.json` for source-build metadata, which only names `main.tex`, `bibliography.bib`, and `icml2026.bst`.

## Paper-to-Code Matches

Because the implementation is inaccessible, I found no paper-to-code matches. The official LaTeX source is internally consistent with the PDF/manuscript claims, but that does not verify implementation behavior.

The paper provides some textual details that would be useful if code existed:

- Hardware: AMD Ryzen 9 7900 CPU and NVIDIA RTX 4090 GPU (`ExperimentalSetup.tex:4`).
- Framework: PyTorch Geometric (`ExperimentalSetup.tex:4`).
- Datasets: Cora, CiteSeer, PubMed with Yang et al. splits (`ExperimentalSetup.tex:12-14`).
- Graph-to-hypergraph conversion: one hyperedge per node containing the node and all graph neighbors (`ExperimentalSetup.tex:16`).
- Model: 3-layer Hypergraph Convolutional Network, hidden dimensions 64 and 32, Leaky ReLU, dropout 0.5, final linear classifier, 200 epochs, learning rate 0.01, weight decay 5e-4, SGD (`ExperimentalSetup.tex:18`).
- Explainer hyperparameter search: learning rates 0.1 and 0.01; momentum values 0, 0.5, 0.9; selected explainer learning rate 0.1 and momentum 0.9 (`ExperimentalSetup.tex:68-71`).
- Evaluation metrics are defined for validity/accuracy, explanation size, and sparsity (`ExperimentalSetup.tex:39-67`).

These details are incomplete without code, exact preprocessing, seeds, run commands, and metric implementations.

## Paper-to-Code Discrepancies and Artifact Problems

1. Code availability claim is not supported by accessible artifacts.
   The manuscript says "The code is publicly available on GitHub" but points to an anonymous 4open URL rather than a `github.com` URL (`ExperimentalSetup.tex:6`). Koala metadata has `github_repo_url: null` and `github_urls: []`. The anonymous URL currently redirects to an API endpoint that returns HTTP 401. This is a direct artifact availability failure.

2. No implementation is available for the proposed dense or sparse variants.
   The paper's core technical claim depends on dense and COO sparse implementations of CF-HyperGNNExplainer (`ExperimentalSetup.tex:4-6`), including a runtime comparison in `Results.tex:97-109`. The official artifacts contain no implementation files, so I cannot verify whether the sparse implementation actually matches the described perturbation operators, whether it recomputes degrees correctly, or whether sparse/dense outputs are equivalent.

3. No preprocessing code exists for the graph-to-hypergraph conversion.
   The paper converts Cora, CiteSeer, and PubMed from graphs to hypergraphs by creating one neighborhood hyperedge per node (`ExperimentalSetup.tex:12-16`). There is no script showing whether citation edges are treated as directed or undirected, whether self-loops are included consistently, how duplicate incidences are handled, or whether PyG dataset transforms preserve the stated Yang et al. splits.

4. No training scripts, checkpoints, or seeds are provided.
   The model training recipe in `ExperimentalSetup.tex:18` is not sufficient to reproduce the trained HGNNs. Missing items include random seeds, initialization, train/validation model selection, early stopping or checkpoint policy, loss function details, device determinism settings, and saved model weights. Since counterfactual validity is measured against trained model predictions, this is a central reproducibility blocker.

5. No explainer optimization script or full hyperparameters are provided.
   `ProblemFormulation.tex:22-50` defines the optimization objective, continuous sigmoid relaxation, threshold 0.5, and distance term, but the artifacts do not specify beta, number of optimization steps, mask initialization distribution, restart policy, stopping criterion, neighborhood radius for V3, or failure handling. The selected explainer learning rate/momentum in `ExperimentalSetup.tex:68-71` is not enough to reproduce Tables 1, 3, or 4.

6. No metric implementation is available for reported tables.
   The formulas in `ExperimentalSetup.tex:39-67` define accuracy, size, and sparsity, but there is no metric code to check edge cases. In particular, the V1 size formula sums over all incidences, while the prose says it is for the node being considered (`ExperimentalSetup.tex:48-52`); without code, it is unclear whether reported sizes count only the target-node row or the full incidence matrix. This ambiguity matters because reported V1 sizes are small and decision-relevant.

7. Baseline evaluation is not auditable.
   The paper compares against CF-GNNExplainer and RCExplainer on original and star-expanded Cora (`ExperimentalSetup.tex:34-36`; `Results.tex:43-60`), but provides no baseline code, versions, commits, parameter settings, graph conversion script, or metric adapter. The fairness and correctness of the graph-vs-hypergraph comparison cannot be checked.

8. Runtime and speedup claims are unsupported.
   `Results.tex:97-109` reports explanation times and 13.5x/13.9x speedups for sparse variants. No timing harness, synchronization policy, number of repeated runs, batch/node sampling protocol, CPU/GPU placement, warmup policy, or raw timings are available. The hardware line alone (`ExperimentalSetup.tex:4`) is insufficient to validate runtime claims.

9. Source comments expose an omitted random-baseline issue.
   `Results.tex:37-39` contains commented-out random baseline rows with higher Cora accuracy than the proposed method (`0.853`, `0.901`, `0.938`) but much worse sparsity/size. `Limitations.tex:7-18` also acknowledges that random perturbation baselines can find many valid counterfactuals and that sparse hypergraphs make V1 near-enumerable. Without code or result logs, I cannot determine whether these omitted controls were run under the same protocol or why they were excluded from the main comparison. This is not direct evidence of a code bug, but it is a material audit concern because validity/accuracy is a headline metric.

## Reproducibility Blockers

- Inaccessible declared code artifact: HTTP 401 and not available through `git ls-remote`.
- No GitHub link in Koala metadata despite the manuscript claiming public GitHub code.
- No runnable implementation of CF-HyperGNNExplainer dense or sparse variants.
- No data preprocessing or hypergraph conversion scripts.
- No training/evaluation entry points.
- No dependency specification or pinned package versions.
- No random seeds or deterministic settings.
- No saved checkpoints or raw outputs.
- No metric implementation or result-generation scripts.
- No baseline implementation details, versions, or commands.
- No timing harness for runtime/speedup claims.

## Claims That Cannot Be Independently Verified

The following claims cannot be verified from the official artifacts because the implementation and result-generation pipeline are absent or inaccessible:

- CF-HyperGNNExplainer V1 and V3 achieve the reported valid-CF rates on Cora, CiteSeer, and PubMed (`Results.tex:7`, `Results.tex:31-36`).
- The explanations have the reported sparsity and explanation sizes (`Results.tex:9`, `Results.tex:31-36`).
- The proposed method outperforms CF-GNNExplainer and RCExplainer under the stated original-graph and star-expanded settings (`Results.tex:11-16`, `Results.tex:56-60`).
- The sparse implementation is 13.5x and 13.9x faster than CF-GNNExplainer in the reported settings (`Results.tex:97-109`).
- The learning-rate and momentum ablations were run under the stated protocol and support selecting alpha=0.1, momentum=0.9 (`ExperimentalSetup.tex:68-71`, `Results.tex:65-93`).
- The implementation correctly applies the perturbation masks, recomputes node/hyperedge degrees after perturbation, thresholds at 0.5, and counts valid counterfactuals as defined (`ProblemFormulation.tex:12-17`, `ProblemFormulation.tex:46-50`, `ProblemFormulation.tex:132-135`).

## Severity for the Acceptance Decision

Severity: high to critical.

The artifact failure materially undermines confidence in the paper. The reported empirical improvements, sparse runtime gains, and ablation conclusions are central to the acceptance case, but none can be reproduced or audited from available official artifacts. The manuscript gives partial prose-level hyperparameters, yet omits the code and operational details required to recover the results. In an implementation audit, this should be treated as weak reproducibility, with a substantial negative score impact unless another role independently obtains the missing code and verifies the reported tables.
