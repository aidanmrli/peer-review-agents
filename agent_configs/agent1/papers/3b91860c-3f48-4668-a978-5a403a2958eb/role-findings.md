# APRIL role findings

## Central claim and reproduction target

APRIL claims a 260K-example Lean proof-repair dataset plus released finetuned models that support the paper's single-shot repair results, especially the headline Qwen3-4B pass@1 improvement from 1.1% to 27.4%.

## Paper and artifact evidence checked

- Read the paper source from the Koala tarball (`tmp/3b91860c/arxiv.tex`).
- Verified the public dataset exists via `https://huggingface.co/api/datasets/uw-math-ai/APRIL`.
- Read the dataset card at `https://huggingface.co/datasets/uw-math-ai/APRIL/raw/main/README.md`.
- Verified both released model cards via:
  - `https://huggingface.co/api/models/uw-math-ai/gAPRIL-w-exp`
  - `https://huggingface.co/api/models/uw-math-ai/gAPRIL-wo-exp`

## Reproducibility result from the smallest meaningful check actually run

Partial support only.

- The paper source says the authors generated **260,125** incorrect proofs from **39,492** compiled theorems and repeatedly frames APRIL as a 260K dataset.
- The released Hugging Face dataset card reports **258,103 examples** with split counts `249,005 / 9,263 / 1,835`, a discrepancy of **2,022 examples** relative to the paper.
- The paper says finetuned models are available at `uw-math-ai/gAPRIL-w-exp` and `uw-math-ai/gAPRIL-wo-exp`.
- Those public model cards identify the released checkpoints as **Goedel-Prover-V2-8B** finetunes (`base_model:Goedel-LM/Goedel-Prover-V2-8B`), not the headline **Qwen3-4B** system emphasized in the abstract and Section 5.

## Implementation or correctness risks

- A dataset-size mismatch of 2,022 examples is not necessarily fatal, but it blocks exact paper-to-release reproduction unless the dropped/filtered examples are documented.
- The currently linked public checkpoints do not let an external reader directly reproduce the headline Qwen3-4B result from the release alone.
- I did not find public artifact text explaining whether the release is a post-paper cleaned subset, nor a note mapping the public Goedel checkpoints to the Qwen headline result.

## Novelty/framing context

This does not attack the dataset contribution itself. It narrows what is presently reproducible from the public artifacts: dataset structure and some Goedel-based ablations appear inspectable, but the flagship Qwen result is not yet obviously reproducible from the linked release.

## Decision impact

Weakens confidence in reproducibility and reporting precision, but not enough by itself to overturn the paper's core dataset contribution. Best use is a public clarification request: explain the 260,125 vs 258,103 mismatch and release or document the exact headline-model artifact/config.
