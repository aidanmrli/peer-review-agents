# Correctness Specialist Report

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Assigned role: Correctness Specialist
Date: 2026-04-26

## Task Scope

Search for critical technical errors in claims, definitions, experiments, metrics, and conclusions.

## Evidence Examined

- `main.tex:52`, `73`, `83`: headline claims of superior sample efficiency/performance.
- `main.tex:118-122`: signed synaptic-weight definition.
- `main.tex:193-222`: imitation learning plus PPO pipeline.
- `main.tex:226-249`: Table 1.
- `main.tex:254-258`: baseline definition.
- `main.tex:267-271`: sample-efficiency and Table 1 interpretation.
- `main.tex:376`: unweighted-graph conclusion.
- `main.tex:422-456`: appendix training details.

## Candidate Error 1: Claimed MLP Comparison Is Not Numerically Reported

Location: `main.tex:52`, `73`, `254-258`, and Table 1 at `main.tex:226-249`.

Issue: The abstract and experiments text say FlyGM is compared against multilayer perceptrons. The main topology table omits the MLP row and reports only three graph models. This prevents verification of the practical claim that biological topology beats a conventional policy architecture.

Severity: major.

Consequence: The strongest machine-learning baseline is not transparently evaluated in the main quantitative evidence. Figure 3 includes an MLP curve visually, but no numeric table, no raw logs, and no seed counts are provided.

## Candidate Error 2: Signed vs Unweighted Connectome Ambiguity

Location: `main.tex:118-122`, `main.tex:256`, and `main.tex:376`.

Issue: The method defines signed synaptic-count weights for FlyGM. The conclusion later states that the method works "even when simplified to an unweighted directed graph without synapse counts or neurotransmitter types." The project page also describes the model as an unweighted directed graph. The paper does not clearly separate which results use signed weights versus unweighted topology.

Severity: major.

Consequence: The central biological-grounding claim changes depending on the implementation. If Table 1 uses unweighted topology, the neurotransmitter-polarity method is not supported by the experiment. If Table 1 uses signed weights, the unweighted conclusion is unsupported.

## Candidate Error 3: Unsupported Statistical Strength

Location: Table 1 and `main.tex:267-271`.

Issue: The paper states that the connectome model has lower error and markedly superior sample efficiency. Table 1 supports a strong angle-error advantage, but not uniform position-error superiority:

```text
high_yaw_angle_reduction=(13.55-8.29)/13.55=38.8%
speed3_yaw0_pos_reduction=(0.0385-0.0364)/0.0385=5.5%
speed3_yaw7_pos_reduction=(0.0370-0.0364)/0.0370=1.6%
```

The rewired graph has lower position error at speed=2,yaw=0 and speed=3,yaw=4. The paper gives mean +/- std but not seed count or statistical tests.

Severity: moderate to major.

Consequence: The broad "lower error" claim should be narrowed to angle stability under turning, not overall tracking superiority.

## Candidate Error 4: PPO/Imitation Specification Is Insufficient for the Claimed Causal Interpretation

Location: `main.tex:193-222`, `main.tex:422-456`.

Issue: The training pipeline contains many hidden choices. The paper does not state seed count, rollout horizon, PPO clip epsilon, discount, GAE lambda, batch size, number of PPO epochs, imitation dataset split, reward definitions, or metric implementation. It also uses an MLP value network (`main.tex:208`), meaning the training signal is not itself connectome-structured.

Severity: moderate.

Consequence: The claim that the connectome topology itself causes the reported advantages is undercontrolled unless the omitted settings and per-seed results are supplied.

## Fatal Errors

No mathematical contradiction that alone falsifies the paper was found. The main correctness issues are experimental support and internal specification, not a proof error.

Confidence level: high for baseline omission and method ambiguity; moderate for causal interpretation because code/raw logs are unavailable.
