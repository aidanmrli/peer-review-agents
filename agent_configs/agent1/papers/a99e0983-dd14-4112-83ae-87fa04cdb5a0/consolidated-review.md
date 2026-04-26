# Consolidated Review

Paper: `a99e0983-dd14-4112-83ae-87fa04cdb5a0`

Title: `Physics-Informed Policy Optimization via Analytic Dynamics Regularization`

Agent: `agent1`

## Bottom line

I could not verify the paper's implementation-level claim from the released artifacts. Two independent passes both stop at the same point: the Koala tarball is source-only, and the paper's reported results depend on a MuJoCo torque-control/ADO/PINN stack that is described in text but not released in executable form.

## Claim being tested

The paper presents `PIPER` as a plug-and-play physics regularizer that:

- adds a differentiable Lagrangian residual to standard actor objectives,
- extracts `M`, `C`, `G`, and contact terms from MuJoCo at runtime,
- trains an auxiliary PINN for acceleration prediction,
- requires no simulator or core-RL modifications,
- improves Fetch-v4 efficiency, precision, and stability.

## Evidence gathered

### Pass 1: source-bundle audit

Commands used:

```bash
curl -L https://koala.science/storage/tarballs/a99e0983-dd14-4112-83ae-87fa04cdb5a0.tar.gz -o tmp/a99e0983.tar.gz
tar -tzf tmp/a99e0983.tar.gz
```

Findings:

- The bundle contains `preprint.tex`, `references.bib`, style files, `00README.json`, and static figures under `Images/`.
- I found no Python code, no MuJoCo XML/URDF assets, no Gymnasium wrapper, no Stable-Baselines/TQC training scripts, no config files, no checkpoints, no logs, and no table-generation outputs.

Consequence:

- The released artifact is sufficient to read the method, but not to execute or independently rebuild the reported training pipeline.

### Pass 2: method-specification audit

Commands used:

```bash
rg -n "(torque|Fetch|PINN|lambda|stability|sigma|MuJoCo)" tmp/a99e0983/preprint.tex
nl -ba tmp/a99e0983/preprint.tex | sed -n '394,559p'
nl -ba tmp/a99e0983/preprint.tex | sed -n '575,711p'
```

Findings:

- The paper explicitly assumes direct joint-torque control: `preprint.tex:394-395`.
- The algorithm also relies on a runtime Automated Dynamics Oracle and a PINN in the optimization loop: `preprint.tex:397-513`.
- The experiments are reported on `FetchReach-v4`, `FetchPush-v4`, `FetchSlide-v4`, and `FetchPickAndPlace-v4`: `preprint.tex:537-549`.
- The implementation section says the method is an additional loss term, but also introduces a `~162k`-parameter PINN and tuned regularization weights: `preprint.tex:549`, `711`.
- Stability is defined as the standard deviation of success rate over the final 100 rollouts (`preprint.tex:558`), yet Table 2 reports non-zero stability values for 100% FetchReach success (`preprint.tex:577-584`).

## Synthesis

The key blocker is not merely "missing code." The paper's central practical claim is that `PIPER` is easy to bolt onto standard RL stacks without simulator changes, but the reported setup appears to require an exact implementation of:

- the MuJoCo oracle queries,
- the PINN architecture and training schedule,
- the torque-control interface used for the Fetch tasks,
- the evaluation and metric computation code.

None of those executable pieces are present in the release. That makes it impossible to determine whether the observed gains come from a minimal regularizer, from unreleased environment/control modifications, or from additional engineering choices not visible in the paper.

## Public-comment payload basis

A concise public comment supported by this file is:

- one-sentence bottom line,
- explicit evidence that the tarball is source-only,
- explicit evidence that the paper assumes direct torque control while evaluating Fetch-v4 tasks,
- explicit statement that two independent passes did not recover the core implementation path,
- one falsifiable question: is there a released wrapper/repo/branch that contains the actual torque-control Fetch setup plus ADO/PINN code?

## Decision impact

This is a material reproducibility limitation. The idea may still be interesting, but I would discount the implementation and empirical claims until the authors release the actual training/evaluation stack or point reviewers to the exact code path that produced these results.
