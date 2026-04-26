# Reproducibility Lead Report

Paper ID: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Assigned role: Reproducibility Lead
Date: 2026-04-25 America/Toronto

## Task Scope

Coordinate the internal review for a reproducibility-first public Koala comment. The central question is whether the paper's reported CF-HyperGNNExplainer empirical claims can be independently reproduced from the official paper, source tarball, and declared artifacts.

## Claims Tested

1. CF-HyperGNNExplainer V1/V3 generate valid, concise counterfactuals on Cora, CiteSeer, and PubMed.
2. The hypergraph-native method outperforms CF-GNNExplainer and RCExplainer on Cora.
3. The sparse implementation is substantially faster than graph-based baselines, with reported 13.5x and 13.9x speedups.
4. The method is reproducible from the publicly available code claimed in the manuscript.

Minimum reproduction target: recover at least one reported table row from official code or, failing that, reconstruct enough preprocessing, training, explainer optimization, and metric calculation from the text to justify a faithful independent implementation.

Tolerance before seeing outcomes:

- Accuracy within 2 percentage points for the same trained model and split.
- Explanation size/sparsity within one reported standard deviation.
- Runtime within a factor of 2 under comparable hardware.
- If no code is available, the paper must specify preprocessing, model training, explainer optimization, seeds, metrics, and baselines well enough for a faithful reimplementation.

## Evidence Examined

- Koala `get_paper` metadata.
- Official PDF/source artifacts under `papers/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be/artifacts/`.
- `ExperimentalSetup.tex`, `ProblemFormulation.tex`, `Results.tex`, `Limitations.tex`, `RelatedWorks.tex`, and `Background.tex`.
- Declared code URL: `https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716`.
- Existing Koala discussion: one outside review by `reviewer-2`, comment `491c7add-ae10-4fef-bfd1-6e2ab58693a9`.

## Role Assignments and Outcomes

- Independent Reproducer A: attempted artifact-first reproduction from official source and declared code. Outcome: blocked; only table arithmetic checks recovered.
- Independent Reproducer B: attempted clean-room reimplementation feasibility from method equations and experimental setup. Outcome: blocked; too many missing optimization, seed, preprocessing, and baseline choices.
- Implementation Auditor: inspected official artifacts and declared code link. Outcome: critical artifact failure; no accessible implementation.
- Correctness Specialist: checked metric definitions, baseline framing, table arithmetic, and source-only random baseline clues. Outcome: major concerns around validity/accuracy interpretation, sparsity denominator, and omitted random baseline context.
- Literature Specialist: checked novelty relative to CF-GNNExplainer, RCExplainer, and hypergraph explainer literature cited by the paper. Outcome: plausible niche contribution, but best framed as a direct CF-GNNExplainer-style adaptation to hypergraph incidence/hyperedge edits, not a broadly validated explainability result.

## Reproducibility Outcome

No central empirical claim was reproduced by two independent roles. Both reproducer roles were blocked before execution because the official implementation is unavailable and the manuscript lacks essential operational details. The two roles agree on the outcome.

What was recovered:

- Cora accuracy deltas in Table 2 are arithmetically consistent: `0.720 - 0.497 = 0.223` and `0.720 - 0.499 = 0.221`.
- Runtime speedups are arithmetically consistent: `44.449 / 3.299 = 13.47` and `44.449 / 3.190 = 13.93`.

What was not recovered:

- Any CF-HyperGNNExplainer run.
- Any trained HGNN model.
- Any Cora/CiteSeer/PubMed hypergraph preprocessing result.
- Any baseline execution.
- Any metric script.
- Any random seed, checkpoint, or raw output.

## Artifact Status

Koala metadata reports `github_repo_url: null` and `github_urls: []`. The manuscript states in `ExperimentalSetup.tex:6` that code is publicly available on GitHub, but the footnote points to `https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716`, not to GitHub. Access checks returned a redirect to `/api/repo/CF-HyperGNNExplainer-0716/file/` followed by HTTP 401, and `git ls-remote` failed with repository not found. The official source tarball contains LaTeX, bibliography/style files, one PNG, `paper.pdf`, and `source.tar.gz`, but no code.

The work is not practically reimplementable from released information. The first missing blockers are the exact graph-to-hypergraph conversion code, trained HGNN setup/seeds, explainer beta/iterations/initialization/restarts/stopping rules, V3 neighborhood radius, baseline adaptation details, and metric code.

## Decision Impact

This is weak reproducibility. Since the contribution is primarily an empirical method paper, missing implementation and incomplete reimplementation details should materially lower confidence. The strongest defensible comment is not that the method is invalid, but that the reported acceptance-case evidence is currently unaudited and should not receive full credit until the code, configurations, seeds, and result-generation scripts are accessible.

Recommended score impact: shift from a weak-accept/positive reading toward weak reject unless the code appears and the tables are independently reproducible.

## Remaining Uncertainty

It remains possible that the authors have a correct private implementation and that the reported tables are accurate. That uncertainty does not rescue the current submission under a reproducibility-first standard because the public artifact link fails and the paper does not contain enough detail for two independent internal roles to rebuild the result.
