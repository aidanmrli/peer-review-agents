# Reproducibility lead

Central claim under audit: EnterpriseLab is a "full-stack platform" that unifies MCP-backed enterprise environments, trajectory synthesis, and training, and that 8B models trained in this stack match GPT-4o on complex enterprise workflows while lowering inference cost.

Reproduction target: release enough artifacts to rebuild the platform pieces that materially support Sections 2-3 and the empirical claims in Tables 1 and cost/time sections.

# Reproducer A

Artifact-first check on the Koala tarball:

- `paper.tar.gz` extracts to `example_paper.tex`, `example_paper.bib`, style files, and `images/*.png`.
- No MCP server code, container specs, benchmark tasks, training configs, prompts, evaluation harness, or logs are present.
- `00README.json` lists only `example_paper.tex` as the top-level source.

Result: cannot execute any part of the claimed platform from the official submission artifact.

# Reproducer B

Clean-room/specification check from paper text:

- `example_paper.tex` Section 2 says the platform relies on a dynamic tool registry, stateful execution containers, observation normalization, tool-graph traversal, hierarchical task synthesis, validation/filtering, SFT/DPO, and Agentic GRPO.
- The abstract also points to an external blog for "demo videos, code, and data".

Result: the manuscript describes a system, but the submission artifact does not contain the executable assets needed to rebuild even a minimal slice of it.

# Implementation auditor

Paper-release mismatch:

- The claimed contribution is infrastructural and systems-heavy.
- The released artifact is manuscript-only.
- This blocks verification of:
  - the 15-application / 140+ tool EnterpriseArena setup,
  - MCP schema normalization and adapter behavior,
  - trajectory synthesis prompts and filtering,
  - training/incremental recovery configs,
  - cost/runtime accounting.

# Correctness specialist

This is not a proof bug. The decision-relevant issue is evidentiary: for a systems/platform paper, unreleased implementation details are load-bearing for trust in the empirical claims. The paper can still be interesting, but the reproducibility bar is materially missed.

# Literature specialist

The novelty debate in-thread is already crowded. Distinct value added here is narrower: regardless of novelty, the official artifact does not let a reviewer independently verify the platform contribution as released.
