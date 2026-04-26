# Literature Specialist Report

Paper ID: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Assigned role: Literature Specialist
Date: 2026-04-25 America/Toronto

## Task Scope

Evaluate whether the novelty and framing are accurate relative to the cited hypergraph explainability and graph counterfactual explanation literature.

## Evidence Examined

- `RelatedWorks.tex:3-9`: hypergraph neural network explainability literature.
- `RelatedWorks.tex:12-18`: counterfactual explanation literature.
- `Background.tex:1-18`: direct relationship to CF-GNNExplainer.
- `Introduction.tex:17-25`: claimed contribution and evaluation framing.
- `bibliography.bib` entries for CF-GNNExplainer, RCExplainer, HyperEX, SHypX, SHy, GNNExplainer, PGExplainer, and related hypergraph neural network work.

## Novelty Claim Checked

The manuscript claims to address the gap that graph counterfactual explainers are not directly applicable to HGNNs because hypergraph structure is encoded by node-hyperedge incidences and hyperedges, not pairwise edges.

## Prior Work Considered

- CF-GNNExplainer: structural counterfactual generation for GNNs through adjacency perturbation.
- RCExplainer: robust graph counterfactual explanations via edge-level changes.
- HyperEX and SHypX: hypergraph explanation/attribution methods for HGNNs.
- SHy and explainable HGNNs for intrinsically interpretable hypergraph models.
- GNNExplainer and PGExplainer: non-counterfactual graph explanation baselines.

## Specific Overlap or Distinction

The novelty is plausible in the narrow counterfactual-hypergraph intersection. `RelatedWorks.tex:4-9` fairly notes that existing hypergraph explainers are primarily sub-hypergraph or node-hyperedge attribution methods, not counterfactual structural-edit methods. `RelatedWorks.tex:17-18` fairly identifies that CF-GNNExplainer and RCExplainer operate over pairwise graph adjacency and therefore do not map cleanly onto incidence/hyperedge edits.

The method is nevertheless a direct adaptation of CF-GNNExplainer's optimization template. `Background.tex:1-18` explicitly states that the approach inherits CF-GNNExplainer's idea and replaces adjacency perturbations with incidence/hyperedge perturbations. This is a legitimate adaptation, but the intellectual contribution should be framed as a targeted extension of a known counterfactual optimization pattern to HGNN incidence structures, not as a broad new theory of explainability.

## Missing Citation or Baseline

The paper cites relevant hypergraph explanation methods. The more important omission is not a missing citation but a missing empirical control: the source and limitations indicate that random or exhaustive perturbation baselines are decision-relevant for sparse neighborhood hypergraphs. A literature-aware framing should include these simple search baselines because for a sparse target-node incidence row, minimal counterfactual search may be close to enumeration.

## Framing Accuracy

Mostly accurate but optimistic:

- Accurate: pairwise graph counterfactual explainers do not directly preserve hyperedge semantics.
- Accurate: incidence-level and hyperedge-level edits are more actionable hypergraph interventions than arbitrary pairwise edge deletion after star expansion.
- Overstated unless further validated: the paper's empirical evidence is sufficient to show that the method identifies "higher-order relations most critical to HGNN decisions" across HGNNs. The evaluation uses citation graphs converted into hypergraphs, one HGNN architecture, and unreleased code.

## Artifact and Reimplementation Feasibility

The literature evaluation is limited by artifact failure. Without the implementation, it is impossible to determine whether the proposed method is a faithful and robust implementation of the stated hypergraph counterfactual idea or a fragile instance of CF-GNNExplainer-style masking.

## Confidence Level

Moderate to high. The core literature distinction is clear from the cited works. The strength of the novelty claim depends heavily on empirical validation, which is not reproducible from available artifacts.

## Consequence for Acceptance

The paper has a plausible niche novelty claim, but the novelty is incremental and acceptance should depend on robust empirical evidence. Because that evidence is currently unreproducible and key random/search controls are not transparently reported, the literature finding does not offset the reproducibility weakness.
