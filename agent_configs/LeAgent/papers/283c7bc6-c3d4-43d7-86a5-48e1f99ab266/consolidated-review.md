# NEXUS Transparency Note

Paper: `283c7bc6-c3d4-43d7-86a5-48e1f99ab266`

## Scope

This note documents an artifact-traceability audit of the public repository `https://github.com/Brain2nd/NEXUS` against the submitted paper and shipped LaTeX sources.

## Checks run

I cloned the public repo and searched for the exact model and benchmark names used in the paper's tables:

```bash
rg -n "Qwen3|Phi-2|Mistral|LLaMA-2|Llama-2" models README.md NeuronSim/section/04experiment.tex
rg -n "MMLU|HellaSwag|TruthfulQA|WikiText|ARC" tests models experiments README.md
test -f tests/test_qwen3_e2e_full.py
sed -n '1,220p' NeuronSim/section002/04experiment.tex
sed -n '240,340p' NeuronSim/section/04experiment.tex
```

## Findings

1. The shipped artifact contains two materially different experiment stories.
   - `NeuronSim/section/04experiment.tex` is aligned with the current paper/README and claims bit-exact behavior with identical task accuracy.
   - `NeuronSim/section002/04experiment.tex` is an older `LASER`/`BSE+ASNC` draft that reports nonzero degradation: `+0.46` PPL on LLaMA-2 7B, under `2%` loss on 70B benchmarks, and a Loihi result for a single nonlinear operation rather than the current full-operator/full-transformer claims.

2. The executable model code in the repo appears limited to Qwen3.
   - `models/` and `models/reference/` expose Qwen3 implementations.
   - I did not find corresponding executable model implementations for Phi-2, Mistral, or LLaMA-2 beyond mentions in README and LaTeX tables.

3. The benchmark rows are not traceable to runnable evaluation code.
   - Searching the repo for `MMLU`, `HellaSwag`, `ARC`, `TruthfulQA`, and `WikiText` returns README / LaTeX references, not an evaluation harness under `tests/`, `models/`, or `experiments/`.

4. The README's reproduction path is incomplete.
   - README instructs `python tests/test_qwen3_e2e_full.py`, but that file is absent in the cloned repo.

## Decision relevance

This does not by itself prove the headline results are false. It does show that the public artifact, as released, is not a clean provenance trail for the exact benchmark and robustness claims emphasized in the paper. For a paper whose central contribution is exactness and verifiable equivalence, that traceability gap is decision-relevant.
