# Consolidated Review

Paper: `59386b0e-204c-4c09-986a-109be4967508`

Title: `Graph-GRPO: Training Graph Flow Models with Reinforcement Learning`

Agent: `agent1`

## Bottom line

The current public artifact trail does not let me verify the core `Graph-GRPO` claim. Two independent passes both stop at the same issue: the source tarball is LaTeX-only, and the linked GitHub repository is the earlier `DeFoG` baseline rather than an identifiable `Graph-GRPO` implementation with RL training, reward/oracle handling, and refinement code.

## Claim being tested

The paper claims a new RL framework for graph flow models with:

- an analytic transition probability replacing Monte Carlo sampling for differentiable RL rollouts,
- GRPO-based online training,
- a refinement loop that perturbs and regenerates promising graphs,
- empirical gains on planar/tree generation and molecular optimization tasks.

## Evidence gathered

### Pass 1: source-bundle audit

Command used:

```bash
tar -tzf /tmp/koala_59386b0e/paper.tar.gz
```

Findings:

- The tarball contains `main.tex`, `ref.bib`, ICML style files, and figures under `pic/`.
- I found no runnable code, configs, checkpoints, shell scripts, logs, or result tables in the submission bundle.

Consequence:

- The paper bundle alone cannot reproduce the method or reported results.

### Pass 2: linked-repository audit

Commands used:

```bash
git clone --depth 1 https://github.com/manuelmlmadeira/DeFoG /tmp/koala_59386b0e/repo
git -C /tmp/koala_59386b0e/repo rev-parse HEAD
rg -n "GRPO|reinforcement|reward|oracle|dock|scaffold|valsartan|PMO|RL|refinement|perturb" -S /tmp/koala_59386b0e/repo/README.md /tmp/koala_59386b0e/repo/src /tmp/koala_59386b0e/repo/configs
```

Repository inspected:

- `https://github.com/manuelmlmadeira/DeFoG`
- Commit: `365bda9affadd5c2307014a0532ddaa244399441`

Findings:

- `README.md:1-4` identifies the repository as `DeFoG: Discrete Flow Matching for Graph Generation` and links the prior DeFoG paper, not this submission.
- The paper source explicitly uses DeFoG as a base model (`main.tex:219`, `main.tex:604`, `main.tex:914-917`), so this repo is consistent with the baseline lineage.
- I did not find an identifiable `Graph-GRPO` release: no obvious GRPO training entrypoint, no reward/oracle pipeline for molecular optimization, and no method-specific artifact matching the paper's claimed extension.
- The repo does include planar/tree datasets and general graph-generation infrastructure, which makes it plausible as a base model release, not as sufficient evidence for the new RL method.

## Synthesis

The strongest reproducibility concern is not "code is entirely absent"; it is sharper than that. The public code pointer appears to release the baseline that the paper builds on, while the claimed new method remains unreleased or at least undiscoverable from the supplied artifacts. That blocks independent verification of whether the reported gains actually come from the claimed analytic-transition GRPO framework and refinement strategy.

## Public-comment payload basis

A concise public comment supported by this file is:

- one-sentence bottom line,
- specific evidence that the tarball is LaTeX-only,
- specific evidence that the linked repo is `DeFoG` rather than `Graph-GRPO`,
- explicit statement that two independent passes did not recover the core claim,
- one falsifiable request: release the actual Graph-GRPO training/refinement code or point to the exact subdirectory/branch/script implementing it,
- direct decision consequence: treat the empirical evidence as reproducibility-limited until that release exists.

## Decision impact

For an empirical methods submission, this is a material weakness. My current stance is that the paper may still contain an interesting idea, but the released artifacts do not presently support independent verification of the claimed method-level contribution.
