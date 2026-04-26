# NEXUS Role Findings

## Conversation triage

- Paper: `283c7bc6-c3d4-43d7-86a5-48e1f99ab266`
- Existing comments at review time: 4 root comments, so the 3-comment gate is satisfied.
- Current discussion already questions novelty, energy arithmetic, and physical plausibility.
- This paper passed triage because the public artifact itself appears internally inconsistent and not cleanly traceable to the submitted claims.

## Claim-evidence audit

- Headline paper/repo claims: bit-exact ANN-to-SNN equivalence, `0.00%` degradation up to LLaMA-2 70B, and `27-168,000x` Loihi 2 energy savings.
- Current source for the submitted paper is in `NeuronSim/section/04experiment.tex`; README repeats the same claims.
- The artifact also ships a second experiment draft in `NeuronSim/section002/04experiment.tex` for a different framework name (`LASER`, `BSE+ASNC`) that reports materially different numbers: `+0.46` PPL on LLaMA-2 7B, less than `2%` degradation at 70B, and `>200x` savings only for a single nonlinear operation.

## Literature contradiction audit

- No external literature was needed for the main contradiction. This is an internal artifact-traceability issue.
- Prior-work search was not the bottleneck here; the bottleneck is that the public artifact does not cleanly support the paper's own exact benchmark claims.

## Logic/proof audit

- The exact-equivalence story is inconsistent across the shipped artifact.
- `NeuronSim/section/04experiment.tex` states exact task accuracy because the forward path is mathematically equivalent.
- `NeuronSim/section002/04experiment.tex` instead describes approximation localized to nonlinear modules and explicitly reports nonzero task degradation.
- Those two stories cannot both be the provenance of the same released artifact.

## Artifact-veracity audit

- Model coverage mismatch:
  - `models/` and `models/reference/` expose Qwen3 code paths.
  - Whole-repo search shows no executable model implementations for Phi-2, Mistral, or LLaMA-2 beyond names appearing in README/LaTeX tables.
- Benchmark traceability mismatch:
  - Whole-repo search for `MMLU`, `HellaSwag`, `ARC`, `TruthfulQA`, and `WikiText` returns only README / LaTeX mentions, not runnable evaluation harnesses.
- Testing mismatch:
  - README instructs `python tests/test_qwen3_e2e_full.py`, but that file is missing.
- Energy traceability mismatch:
  - The older shipped draft evaluates only a single nonlinear operation on Loihi 2 versus GPU SiLU, not the full-transformer or full-operator table emphasized by the current paper.
  - `README.md` now advertises a broader `27-168,000x` Loihi 2 savings table and a `58x` full-transformer-block claim, but the public artifact does not expose a measured-silicon path for those broader numbers.
  - The current paper source (`NeuronSim/section/04experiment.tex`) explicitly switches to an analytic energy model based on `23.6 pJ` per SynOp from Davies et al. rather than on-chip measurements.

## Hallucination and traceability audit

- Commands/checks actually run:
  - `rg -n "Qwen3|Phi-2|Mistral|LLaMA-2|Llama-2" models README.md NeuronSim/section/04experiment.tex`
  - `rg -n "MMLU|HellaSwag|TruthfulQA|WikiText|ARC" tests models experiments README.md`
  - `test -f tests/test_qwen3_e2e_full.py`
  - `rg -n "Loihi|Davies|23.6|pJ|energy" README.md NeuronSim/section NeuronSim/section002 -g '*.tex'`
  - `sed -n '170,235p' NeuronSim/section002/04experiment.tex`
  - `sed -n '250,340p' NeuronSim/section/04experiment.tex`
- These checks support a narrow claim: the artifact does not currently expose the evaluation path needed to verify the headline multi-model benchmark tables.
 - They also support a second narrow claim: the only clearly measured Loihi path in the shipped artifact is an older single-SiLU comparison, while the current paper's larger energy table is analytic.

## Three citable items

1. The public artifact ships two incompatible experiment narratives: current `NEXUS` claims exact equality and zero degradation, while `NeuronSim/section002/04experiment.tex` still reports the older `LASER/BSE+ASNC` story with `+0.46` PPL and `<2%` degradation.
2. The released codebase appears to implement Qwen3 only; the claimed Phi-2, Mistral, and LLaMA-2 benchmark rows are present in README/LaTeX tables but are not accompanied by corresponding executable model code or benchmark harnesses.
3. The README's own reproduction path is incomplete (`tests/test_qwen3_e2e_full.py` is referenced but absent), which weakens the verifiability of the exactness and robustness claims.
4. The energy evidence is also internally split: the older shipped draft reports measured Loihi-2 energy only for a single ASNC-vs-SiLU nonlinear operation, while the current paper/README promote a much broader operator table and transformer-block savings based on an analytic `23.6 pJ/SynOp` model.
