# Independent Reproducer B Report

Paper ID: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Assigned role: Independent Reproducer B
Date: 2026-04-25 America/Toronto

## Task Scope

Independently test whether the main empirical claim can be validated by a clean-room reconstruction route, without relying on Independent Reproducer A's report.

## Claim Attempted

Validate the claim that CF-HyperGNNExplainer finds valid and sparse counterfactual explanations on citation-network hypergraphs and improves over graph-native counterfactual explainers.

## Independent Route Used

I read the method and metric definitions and attempted to specify the smallest faithful implementation:

1. Load Cora/CiteSeer/PubMed with Yang et al. splits.
2. Convert each graph to a hypergraph by creating one hyperedge per node containing the node and all graph neighbors.
3. Train the stated 3-layer HGNN.
4. Freeze the model and optimize the V1 or V3 mask.
5. Threshold the mask at 0.5.
6. Evaluate validity, explanation size, and sparsity on the test split.

## Evidence Examined

- `ProblemFormulation.tex:12-17`: perturbed operator with incidence mask and degree recomputation.
- `ProblemFormulation.tex:22-50`: loss, log-probability term, L1 distance, indicator term, continuous relaxation, threshold 0.5.
- `ProblemFormulation.tex:58-83`: V1 node-hyperedge perturbation.
- `ProblemFormulation.tex:115-135`: V3 hyperedge perturbation and degree recomputation.
- `ExperimentalSetup.tex:12-18`: dataset conversion and HGNN architecture/training recipe.
- `ExperimentalSetup.tex:39-67`: metrics.
- `ExperimentalSetup.tex:68-71`: explainer learning-rate/momentum search.

## Commands, Calculations, and Trace Steps

I used source inspection rather than execution. The clean-room implementation failed at the following underdetermined decisions:

- Whether citation graph edges are directed or symmetrized before neighborhood hyperedge construction.
- Whether self-incidence is always included and whether duplicate incidences are removed.
- Exact PyG dataset class/version and split handling.
- Hyperedge weights.
- HGNN layer implementation variant and whether PyG's `HypergraphConv` is used or a custom dense/sparse operator.
- Training loss, model selection, random seeds, initialization, and checkpoint policy.
- Explainer beta value, number of mask optimization steps, initialization, restart policy, and failure criterion.
- V3 neighborhood radius `n`.
- Whether V1 explanation size is counted over the target-node incidence row or the full incidence matrix after masking.
- Exact adaptation and hyperparameters for CF-GNNExplainer and RCExplainer on original and star-expanded Cora.

## Observed Result

No executable reproduction can be defined without introducing reviewer assumptions. The paper provides a mathematical template but not an operational protocol. A clean-room implementation would be a plausible reimplementation, not a reproduction.

The source includes a decision-relevant clue: `Results.tex:37-39` contains commented-out random-baseline rows with Cora accuracies of `0.853`, `0.901`, and `0.938`, all above the reported proposed-method Cora accuracies (`0.720` for V1 and `0.647` for V3). The limitations section also acknowledges that random perturbation can find many valid counterfactuals in sparse hypergraphs. This makes the exact random-baseline protocol and metric implementation essential for interpreting the validity/accuracy claim.

## Match Status

Blocked. I could not independently recover any central table row.

## Agreement with Reproducer A

After completing this independent route, my outcome agrees with Reproducer A: the central empirical claims are not reproducible from the official artifacts. Reproducer A established artifact/code inaccessibility; this pass independently establishes that the paper text is not sufficient for faithful reimplementation.

## Limitations

I did not run a synthetic toy implementation because it would not test the reported claims. It would only demonstrate that one possible interpretation of the method can be coded.

## Confidence Level

High confidence that a faithful reproduction is blocked. High confidence that the random-baseline and metric details are decision-relevant.

## Decision Impact

The acceptance case depends on unreproduced empirical tables. The paper should be materially marked down until runnable code, configs, seeds, and full baseline protocols are released.
