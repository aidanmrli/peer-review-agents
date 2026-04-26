# Correctness Specialist Report

Paper ID: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Assigned role: Correctness Specialist
Date: 2026-04-25 America/Toronto

## Task Scope

Check for technical errors or unsupported conclusions in metric definitions, baselines, ablations, and result interpretation.

## Evidence Examined

- `ExperimentalSetup.tex:39-67`: accuracy, explanation size, and sparsity metrics.
- `ExperimentalSetup.tex:34-36`: graph-based baseline setup and star expansion.
- `ExperimentalSetup.tex:68-71`: hyperparameter selection.
- `Results.tex:7-16`, `Results.tex:31-60`, `Results.tex:65-109`: main result tables and claims.
- `Results.tex:37-39`: commented-out random baseline rows in the source.
- `Limitations.tex:2-18`: sparse hypergraph/random baseline limitations.

## Findings

### 1. Validity/accuracy is weak as a headline metric

Candidate error: The paper foregrounds "accuracy" as the proportion of test nodes for which any prediction flip is found, but this does not establish explanation faithfulness or superiority over simple search/random perturbation.

Location: `ExperimentalSetup.tex:39-46`; `Results.tex:7-16`; `Results.tex:31-36`.

Evidence: The paper's own source comments contain random baseline rows with higher Cora accuracy than the proposed method:

- random V1: `0.853` versus CF-HyperGNNExplainer V1 `0.720`
- random V3: `0.901` versus CF-HyperGNNExplainer V3 `0.647`
- random full: `0.938`

`Limitations.tex:7-18` explicitly acknowledges that random perturbation baselines can find many valid counterfactuals and that sparse hypergraphs can make minimal counterfactuals easy to sample or enumerate.

Severity: major.

Consequence: The reported valid-CF rate is not enough to support the main empirical interpretation unless paired with a transparent random/exhaustive baseline and a stronger faithfulness/minimality analysis.

### 2. Sparsity is likely inflated by a global denominator

Candidate error: The sparsity formulas use the full incidence count or the full hyperedge count as denominator, even though explanations are local to a target node or target neighborhood.

Location: `ExperimentalSetup.tex:60-67`; PubMed sparsity in `Results.tex:33` and `Results.tex:36`.

Evidence: PubMed reports sparsity near `0.999` for both variants while explanation sizes are around 2.5-2.9. If sparsity is normalized by a large global graph/hypergraph denominator, the metric will approach one almost mechanically for local explanations in large graphs. This makes cross-dataset sparsity comparisons potentially misleading: PubMed can look more sparse because the denominator is much larger, not because explanations are more locally minimal.

Severity: major for interpretation, moderate for formula correctness.

Consequence: The claim that explanations are "highly sparse" across datasets is not a robust local explanation-quality claim without a local-denominator metric.

### 3. Baseline comparison is under-specified and potentially confounded

Candidate error: The graph-based baseline comparison mixes original Cora graph and star-expanded hypergraph-to-graph conversion, but the protocol is not sufficiently specified to establish fairness.

Location: `ExperimentalSetup.tex:34-36`; `Results.tex:43-60`.

Evidence: Star expansion doubles nodes and changes graph topology. CF-GNNExplainer and RCExplainer operate on different structural primitives than the proposed incidence/hyperedge removals. The paper does not specify baseline hyperparameters, code versions, budgets, stopping rules, or whether explanation size/sparsity are computed under directly comparable denominators. No code is available to audit the adaptation.

Severity: major.

Consequence: The Cora superiority claim may be true, but its fairness cannot be verified from the paper.

### 4. Runtime speedup arithmetic is correct but runtime methodology is not auditable

Candidate error: The arithmetic for speedups is consistent, but the runtime conclusion lacks enough methodology to be decision-grade.

Location: `Results.tex:97-109`.

Evidence:

- `44.449 / 3.299 = 13.47`, matching 13.5x after rounding.
- `44.449 / 3.190 = 13.93`, matching 13.9x after rounding.

However, no timing harness, repeated-run statistics, GPU synchronization policy, node sampling protocol, warmup, or code is provided.

Severity: moderate.

Consequence: The speedup table is arithmetically coherent but not independently auditable.

### 5. Hyperparameter selection lacks sufficient protocol detail

Candidate error: The paper selects explainer learning rate `0.1` and momentum `0.9` from Cora-only ablations but does not specify other optimization knobs.

Location: `ExperimentalSetup.tex:68-71`; `Results.tex:65-93`.

Evidence: Missing beta, number of steps, initialization, stopping rules, restarts, and failure handling. These choices can trade off validity, sparsity, and explanation size.

Severity: major for reproducibility, moderate for correctness.

Consequence: The ablation is not enough to support a reproducible default configuration.

## GitHub/Artifact Status

No accessible implementation was available. This prevents checking whether the paper's equations, metrics, and tables match actual code.

## Confidence Level

High confidence in the artifact and metric interpretation concerns. Moderate confidence in baseline-confounding severity because code might clarify details if released.

## Decision Impact

The correctness issues do not prove the method is wrong, but they substantially weaken the empirical acceptance case. The paper needs artifact release plus revised metric/baseline reporting before its central claims should receive strong confidence.
