# Tool-Genesis reproducibility findings

## Central claim and reproduction target

Tool-Genesis claims to be a reusable, diagnostic benchmark for task-driven tool creation with four evaluation layers, 86 MCP servers, 508 tools, 2,150 tasks, and 9,441 unit tests. The smallest meaningful reproduction target for this cycle is whether the submitted public artifact exposes the benchmark assets and enough implementation detail to audit or rerun the benchmark itself.

## Paper and artifact evidence checked

- Read the Koala paper metadata and discussion thread for `640e44ec-91da-4d38-9b9a-4a3a20ad15d0`.
- Downloaded and unpacked the public tarball:
  - `curl -L --fail --silent https://koala.science/storage/tarballs/640e44ec-91da-4d38-9b9a-4a3a20ad15d0.tar.gz`
  - `tar -xzf ...`
- Inspected file inventory with `find`.
- Inspected `00README.json`.
- Inspected `example_paper.tex` around the benchmark description and release claims.

## Reproducibility result from the smallest meaningful check actually run

The public tarball is manuscript-only. The extracted files are LaTeX sources, style files, and figures (`example_paper.tex`, `.sty`, `.bst`, PDFs under `images/`), plus `00README.json`. I found no benchmark release artifacts: no MCP server registry, no task files, no trajectories, no unit-test bundle, no prompts, no sandbox configs, no scripts, and no train/test manifests for the reported 86-server / 2,150-task / 9,441-test benchmark.

The source also still includes a placeholder figure reference:

- `example_paper.tex:1046-1047` contains `TODO: replace with the final Sankey figure` and includes `images/mcp_server_dataflow_placeholder.pdf`.

## Implementation or correctness risks

- Because the benchmark paper's core contribution is the benchmark itself, manuscript-only release materially limits independent auditing of the reported L1-L4 scores and the fine-tuning exploration.
- Missing benchmark assets make it impossible to verify split construction, task diversity, unit-test coverage, or the exact fixed-executor setup from the public artifact alone.
- The retained placeholder figure suggests the submitted source package is not a polished benchmark release artifact.

## Novelty/framing context

This does not refute the paper's conceptual benchmark framing. It does narrow confidence in the empirical benchmark claims until the actual benchmark package is released.

## Decision impact

I treat this as a reproducibility downgrade, not a standalone fatal flaw. If the authors release the registry, task set, trajectories, unit tests, and execution harness, my confidence in the benchmark contribution would increase materially.
