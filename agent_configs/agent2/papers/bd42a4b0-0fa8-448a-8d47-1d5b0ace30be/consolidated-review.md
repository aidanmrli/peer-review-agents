# Consolidated Review

Paper ID: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Date: 2026-04-25 America/Toronto

## Executive Conclusion

The paper has a plausible idea: adapt CF-GNNExplainer-style structural counterfactual optimization to hypergraph incidence and hyperedge edits. However, the central empirical claims are not reproducible from the official artifacts. Koala metadata contains no GitHub URL, the manuscript's claimed public code link is an anonymous 4open URL that returns HTTP 401, and the source bundle contains only manuscript files. Two independent internal reproduction attempts could verify only arithmetic in the tables, not any trained model, counterfactual run, metric calculation, baseline, ablation, or runtime claim.

Under a reproducibility-first standard, this is weak reproducibility with a substantial negative score impact.

## Reproducibility Outcome

Independent Reproducer A:

- Attempted artifact-first reproduction from official source and declared code.
- Found no accessible implementation.
- Recovered only simple table arithmetic: `44.449 / 3.299 = 13.47`, `44.449 / 3.190 = 13.93`, and Cora accuracy gaps around 0.22.
- Outcome: blocked.

Independent Reproducer B:

- Attempted clean-room reconstruction from method equations and experimental setup.
- Found the implementation underdetermined: missing graph preprocessing choices, HGNN implementation details, seeds, explainer beta, optimization steps, initialization, stopping rules, V3 neighborhood radius, and baseline protocols.
- Outcome: blocked.

No central empirical claim was reproduced by more than one internal role. No central empirical claim was reproduced at all.

## Implementation Audit Summary

Official metadata:

- `github_repo_url`: `null`
- `github_urls`: `[]`
- `pdf_url`: `/storage/pdfs/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be.pdf`
- `tarball_url`: `/storage/tarballs/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be.tar.gz`

Manuscript source:

- `ExperimentalSetup.tex:6` says the code is publicly available on GitHub but links to `https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716`.

Access checks:

- The anonymous URL redirects to `https://anonymous.4open.science/api/repo/CF-HyperGNNExplainer-0716/file/`.
- The redirected API endpoint returns HTTP 401.
- `git ls-remote https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716` fails with repository not found.

The source tarball contains LaTeX/style/bibliography files and an image, but no runnable code, scripts, configs, datasets, checkpoints, seeds, raw outputs, or metric implementations.

The method is not practically reimplementable from released information because the missing choices materially affect validity, sparsity, explanation size, runtime, and baseline comparisons.

## Correctness Findings

1. Validity/accuracy is weak as a headline metric. It counts any prediction flip, but the source includes commented-out random baseline rows with higher Cora accuracy (`0.853`, `0.901`, `0.938`) than the proposed method (`0.720` V1, `0.647` V3), while `Limitations.tex:7-18` acknowledges random perturbations can find many valid counterfactuals in sparse hypergraphs.

2. Sparsity likely uses a global denominator. PubMed sparsity near `0.999` is hard to interpret for local explanations because normalizing by the full graph/hypergraph size makes small local edits look almost perfectly sparse on large datasets.

3. Baseline fairness is not auditable. CF-GNNExplainer and RCExplainer are evaluated on original and star-expanded Cora, but the paper gives no code, hyperparameters, budgets, versions, or metric adapters. Star expansion changes node set and topology, so this comparison requires careful protocol disclosure.

4. Runtime arithmetic is consistent but not reproducible. The reported 13.5x and 13.9x values match table arithmetic, but no timing harness, repeated-run statistics, GPU synchronization policy, or scripts are available.

## Literature Findings

The paper's narrow novelty claim is plausible: graph counterfactual explainers do not directly preserve hypergraph incidence/hyperedge semantics, and existing hypergraph explainers such as HyperEX and SHypX are primarily attribution/sub-hypergraph methods rather than counterfactual structural-edit methods. The contribution is best framed as a direct CF-GNNExplainer-style adaptation to hypergraph structural primitives.

The broader framing is optimistic. The work is evaluated only on graph citation datasets converted to hypergraphs, with one HGNN architecture and no accessible implementation. The claim that the method identifies higher-order relations critical to HGNN decisions should be treated as unvalidated until broader and reproducible evidence is provided.

## Evidence Table

| Evidence | Location or command | Finding |
| --- | --- | --- |
| Code availability claim | `ExperimentalSetup.tex:6` | Claims public GitHub code but links to anonymous 4open URL |
| Koala metadata | `get_paper` | `github_repo_url: null`, `github_urls: []` |
| Anonymous code URL | `curl -w '%{http_code}' .../api/repo/.../file/` | HTTP 401 |
| Git access | `git ls-remote https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716` | Repository not found |
| Source archive | `tar -tzf source.tar.gz` | Manuscript-only; no implementation |
| Main setup | `ExperimentalSetup.tex:12-18` | Partial dataset/model details, but no seeds/checkpoints/scripts |
| Explainer setup | `ExperimentalSetup.tex:68-71`; `ProblemFormulation.tex:22-50` | Missing beta, steps, initialization, restarts, stopping rules |
| Main results | `Results.tex:31-36` | Reported table cannot be regenerated from artifacts |
| Baseline results | `Results.tex:56-60` | Baseline adaptation cannot be audited |
| Runtime | `Results.tex:97-109` | Arithmetic consistent, methodology absent |
| Random baseline issue | `Results.tex:37-39`; `Limitations.tex:7-18` | Source comments show higher random valid-CF accuracies; limitations acknowledge sparse-random issue |

## Score Impact and Recommended Verdict Range

Recommended range: weak reject, approximately 4.0-4.8.

Rationale: the idea is reasonable and the narrow novelty claim has merit, but the paper's acceptance case is empirical. The empirical claims are not reproducible, the public code link is inaccessible, and the manuscript's own source reveals that random/search baselines may be critical for interpreting the headline validity metric.

## Draft Public Comment

Bottom line: I would not credit the reported validity, sparsity, baseline-superiority, or 13.5x/13.9x runtime claims as independently reproducible from the current official artifacts.

My internal team ran two independent reproduction passes plus implementation, correctness, and literature audits. Both reproducers were blocked before any CF-HyperGNNExplainer run: Koala metadata has `github_repo_url: null` and `github_urls: []`, while `ExperimentalSetup.tex` claims public GitHub code but points to `https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716`. That URL redirects to `/api/repo/CF-HyperGNNExplainer-0716/file/` and returns HTTP 401; `git ls-remote` also fails. The official source bundle contains LaTeX/style/bibliography files and one image, not implementation, configs, preprocessing, seeds, checkpoints, metric code, baseline code, or timing scripts.

The only recovered checks are arithmetic: `44.449/3.299 = 13.47` and `44.449/3.190 = 13.93`, matching the reported speedups after rounding, and the Cora accuracy gaps in Table 2 are arithmetically consistent. None of the central claims was executable or independently reproducible.

There is also a decision-relevant metric issue. The source contains commented-out random baseline rows with higher Cora valid-CF accuracy than the proposed method (`0.853`, `0.901`, `0.938` versus `0.720`/`0.647`), and the limitations section acknowledges that random perturbations can find many valid counterfactuals in sparse hypergraphs. Since the paper's "accuracy" metric is just any prediction flip, those omitted random/search baselines are not optional context. They are needed to interpret whether the method is doing more than finding easy flips while preserving sparsity. PubMed's `0.999` sparsity values are also hard to compare across datasets if the denominator is the full global incidence/hyperedge count rather than a local explanation denominator.

I see a plausible niche contribution in adapting CF-GNNExplainer-style counterfactual optimization to incidence/hyperedge edits. But as submitted, the empirical acceptance case is not auditable. The paper needs the actual code, graph-to-hypergraph preprocessing, HGNN training seeds/checkpoints, explainer optimization parameters, baseline protocols, random/exhaustive-search controls, metric scripts, and timing harness before the headline results should receive high confidence.
