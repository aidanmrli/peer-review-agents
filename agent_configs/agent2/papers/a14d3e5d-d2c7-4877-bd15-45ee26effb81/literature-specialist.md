# Literature Specialist Report

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Assigned role: Literature Specialist
Date: 2026-04-26

## Task Scope

Evaluate whether the novelty and framing are supported by permitted prior work available before or at the paper release. I used the paper's own bibliography and manuscript discussion, not later reviews, decisions, citations, social media, or post-publication commentary about the same paper.

## Prior Work Considered

Connectomics and connectome-constrained modeling:

- Pospisil et al. 2024, "The Fly Connectome Reveals a Path to the Effectome" (`refs.bib:1-13`): motivates linear-dynamical connectome/effectome modeling and is explicitly used for the fixed recurrent operator idea.
- Dorkenwald et al. 2024, "Neuronal Wiring Diagram of an Adult Brain" (`refs.bib:78-92`): provides the adult fly whole-brain connectome resource.
- Azevedo et al. 2024 and Lesser et al. 2024 (`refs.bib:212-242`): ventral nerve cord and leg/wing premotor network reconstructions.
- Lappalainen et al. 2024, "Connectome-Constrained Networks Predict Neural Activity across the Fly Visual System" (`refs.bib:261-274`): shows connectome-constrained neural activity prediction in the fly visual system.
- Shiu et al. 2024, "A Drosophila Computational Brain Model Reveals Sensorimotor Processing" (`refs.bib:276-291`): prior Drosophila computational brain model for sensorimotor processing.

Embodied fly simulation:

- Lobato-Rios et al. 2022, NeuroMechFly (`refs.bib:363-374`).
- Wang-Chen et al. 2024, NeuroMechFly v2 (`refs.bib:376-389`).
- Vaxenburg et al. 2025, flybody (`refs.bib:391-404`).

## Novelty Claim Checked

The paper claims novelty in directly instantiating a whole-brain Drosophila connectome as a graph neural policy for whole-body locomotion control in flybody, rather than studying a restricted subsystem or using a generic MLP controller.

## Overlap and Distinction

The high-level novelty is plausible. Prior work supplies the connectome, biological simulator, and connectome-constrained modeling precedent, but the paper's combination of whole-brain graph topology with RL/IL policy learning for whole-body fly locomotion appears distinct within the cited literature.

The distinction should be framed carefully:

- FlyWire/flybody are substantial external resources. The paper's contribution is not the simulator or connectome dataset.
- The training pipeline, imitation learning plus PPO, is standard. The novelty is the topology prior and its integration with flybody.
- The paper's related-work statement at `main.tex:88-90` is broadly accurate in saying prior connectome-constrained work focused on neural activity or restricted subsystems, while flybody/NeuroMechFly controllers were not whole-brain-connectome policy architectures.

## Missing or Underemphasized Comparisons

The most decision-relevant missing comparison is not a citation but a baseline/result presentation gap: the paper cites and describes generic MLP controllers, but Table 1 does not give the MLP numerical results. Since flybody provides MLP controllers and the paper's own pipeline begins from MLP expert trajectories, the MLP comparison is essential for the practical novelty claim.

The paper also should distinguish "signed synaptic-count connectome controller" from "unweighted topology controller." Prior effectome-style work motivates signed/weighted operators, while the paper's conclusion and project page talk about an unweighted directed graph. That ambiguity weakens the literature-grounding of the claimed biological fidelity.

## Consequence for Acceptance

Literature novelty is a relative strength. It does not offset the reproducibility failure because the central empirical claim is not independently verifiable. If the code and exact experimental pipeline are released, the paper could be a valuable empirical bridge between connectomics and embodied RL.

Confidence level: moderate to high.
