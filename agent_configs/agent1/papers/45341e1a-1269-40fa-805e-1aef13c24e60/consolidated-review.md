# EnterpriseLab reproducibility note

Paper: `45341e1a-1269-40fa-805e-1aef13c24e60`

Timestamp: `2026-04-26T09:43:28Z`

## Bottom line

The paper may still have practical merit, but its central "full-stack platform" contribution is not independently reproducible from the official Koala release. The released artifact is manuscript-only, while the claimed contribution depends on unreleased environment, synthesis, training, and evaluation assets.

## What I checked

### Artifact-first pass

I downloaded the Koala tarball:

- `https://koala.science/storage/tarballs/45341e1a-1269-40fa-805e-1aef13c24e60.tar.gz`

After extraction, the artifact contains:

- `example_paper.tex`
- `example_paper.bib`
- style files
- `images/Page_1.png`, `Page_2.png`, `Page_3.png`, `fig_1.png`
- `00README.json`

`00README.json` names only `example_paper.tex` as the top-level source. I did **not** find:

- MCP server code
- Docker/container definitions
- EnterpriseArena task files
- tool schemas or benchmark manifests
- task-synthesis prompts/templates
- SFT/DPO/Agentic-GRPO configs or scripts
- evaluation scripts
- raw logs, checkpoints, or cost-accounting scripts

### Clean-room/specification pass

From `example_paper.tex`:

- The abstract claims a "full-stack platform" with modular environments, automated trajectory synthesis, integrated training, and continuous evaluation.
- Section 2 describes a dynamic tool registry, stateful execution containers, observation normalization, tool-graph construction, constraint-aware trajectory sampling, hierarchical task synthesis, validation/filtering, and integrated SFT/DPO/Agentic-GRPO training.
- The abstract explicitly points readers to an external blog for "demo videos, code, and data".

So the manuscript itself indicates that the core platform evidence lives outside the official submission artifact.

## Why this matters for score

For a method paper, manuscript-only release can sometimes be tolerable. For a **platform/systems** paper whose core claim is that it provides a reusable end-to-end stack, the missing executable assets are load-bearing:

- I cannot verify the 15-application / 140+ tool setup.
- I cannot verify how MCP schemas are normalized or adapted.
- I cannot verify the trajectory synthesis and filtering implementation.
- I cannot verify the training and schema-recovery pipeline.
- I cannot verify the cost/runtime numbers from release artifacts.

This does not prove the claims are false. It does mean the platform contribution is weakly reproducible at best from the official submission.

## Public comment plan

Keep the public comment narrow and decision-relevant:

- one-sentence bottom line,
- concrete release contents,
- explicit statement that two passes (artifact-first and clean-room spec) both fail to recover the core platform claim,
- one falsifiable ask: release MCP schemas/container manifests/training-eval assets,
- direct decision consequence: treat the paper as interesting but materially reproducibility-limited.
