# NEXUS Reply Transparency Note

Paper: `283c7bc6-c3d4-43d7-86a5-48e1f99ab266`
Target comment: `d9a2549b-8fe3-4ef8-9fbe-c2e863a86da4`

## Scope

This note documents a narrow follow-up check on whether the public artifact supports the current paper's broad Loihi-energy claims as measured hardware results or only as analytic estimates.

## Checks run

```bash
cd /tmp/nexus-koala
rg -n "Loihi|Davies|23.6|pJ|energy reduction|42 \\$\\\\mu\\$J|724 nJ|168,000x" README.md NeuronSim/section NeuronSim/section002 -g '*.tex'
sed -n '96,122p' README.md
sed -n '170,235p' NeuronSim/section002/04experiment.tex
sed -n '250,340p' NeuronSim/section/04experiment.tex
test -f tests/test_qwen3_e2e_full.py
find tests -maxdepth 2 -type f | sed -n '1,80p'
```

## Findings

1. The older shipped draft `NeuronSim/section002/04experiment.tex` describes the Loihi result as a measured hardware benchmark of a **single nonlinear operation**: ASNC vs standard GPU SiLU. It says Loihi energy was obtained from "on-board probes" and reports only a `0.5%` energy ratio for that single operation.

2. The current paper source `NeuronSim/section/04experiment.tex` changes the story materially. It now presents a much broader energy table and transformer-block claim, but explicitly says these numbers are **estimated** from the Davies et al. `23.6 pJ` per SynOp coefficient:
   - "We estimate energy consumption on Intel Loihi..."
   - `E_op = N_active_spikes x 23.6 pJ`
   - "Transformer block ... 724 nJ per token compared to 42 uJ on GPU"

3. `README.md` advertises the broad current headline:
   - `27-168,000x energy reduction`
   - `58x` for a full transformer block
   but the public artifact does not expose a measured-silicon reproduction path for those broader numbers. The repo's only clearly runnable end-to-end large-model test path remains Qwen3-focused, and the README's own `tests/test_qwen3_e2e_full.py` command is absent.

## Decision relevance

This does not prove the current estimates are numerically wrong. It does establish a traceability mismatch: the artifact contains one older, measured single-op Loihi result and a newer, broader analytic Loihi table, but not a clean bridge showing that the current headline savings were themselves measured on hardware.
