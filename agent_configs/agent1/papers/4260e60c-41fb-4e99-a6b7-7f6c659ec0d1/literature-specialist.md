# Literature Specialist Report

Paper: 4260e60c-41fb-4e99-a6b7-7f6c659ec0d1, "Demystifying When Pruning Works via Representation Hierarchies"

Role: Literature Specialist

## Novelty Claim Checked

The paper's core novelty claim is not a new pruning algorithm. The claim checked here is that the paper provides a representation-level explanation for when pruning works: pruning perturbations remain small in embedding and logit spaces, but the softmax probability space amplifies logit perturbations, causing autoregressive generation to degrade while retrieval and multiple-choice classification can remain stable in lower-dimensional or task-specific subspaces.

This claim appears in `artifacts/main.tex` abstract and conclusion, `artifacts/sections/introduction.tex`, `artifacts/sections/method.tex`, and `artifacts/sections/experiments.tex`. The paper explicitly decomposes the pipeline into embedding, logit, and probability spaces; argues that the LM head attenuates relative orthogonal perturbations; derives second-order softmax/KL approximations; and uses subspace behavior to explain multiple-choice robustness.

## Prior Work Considered

- Wanda: Sun et al., "A Simple and Effective Pruning Approach for Large Language Models", arXiv:2306.11695 / ICLR 2024. This is a training-free weight sparsification method using weight magnitudes multiplied by input activations. It reports pruning performance across language benchmarks and is used by the submitted paper as an intra-layer pruning baseline.
- SparseGPT: Frantar and Alistarh, "SparseGPT: Massive Language Models Can Be Accurately Pruned in One-Shot", arXiv:2301.00774 / ICML 2023. This is a one-shot second-order pruning method for GPT-family models, reporting 50-60 percent sparsity with small perplexity or zero-shot accuracy loss. It is used as another intra-layer baseline.
- ShortGPT: Men et al., "ShortGPT: Layers in Large Language Models are More Redundant Than You Expect", arXiv:2403.03853 / ACL Findings 2025. This work identifies layer redundancy with Block Influence, based on input-output similarity, and directly removes low-influence blocks. It is used as a block-dropping baseline.
- Gromov et al., "The Unreasonable Ineffectiveness of the Deeper Layers", arXiv:2403.17887. This work prunes blocks of layers by similarity across layers and then heals with finetuning. It strongly overlaps with the submitted paper's hidden-state robustness premise.
- He et al., "Uncovering the Redundancy in Transformers via a Unified Study of Layer Dropping", TMLR 2026 / LLM-Drop. This prior work studies block, MLP, and attention layer dropping and reports substantial attention-layer redundancy, with code at CASE-Lab-UMD/LLM-Drop. The submitted paper directly builds on this layer-dropping line and cites it for the non-uniform effectiveness of pruning.
- LLM-Pruner: Ma et al., "LLM-Pruner: On the Structural Pruning of Large Language Models", arXiv:2305.11627 / NeurIPS 2023. This is a structural pruning method with gradient-based group selection and LoRA recovery, and it explicitly evaluates zero-shot classification and generation preservation.
- LayerSkip: Elhoushi et al., "LayerSkip: Enabling Early Exit Inference and Self-Speculative Decoding", arXiv:2404.16710 / ACL 2024. This is not a pruning baseline, but it is a relevant layer-dropout and early-exit prior showing that generation from shallower exits can be made useful when trained and verified.
- SliceGPT: Ashkboos et al., "SliceGPT: Compress Large Language Models by Deleting Rows and Columns", arXiv:2401.15024. This is a post-training structural compression method that deletes rows/columns and preserves much zero-shot performance; it is peripheral but relevant to broad claims about structural LLM compression.

## Specific Overlap or Distinction

The submitted paper substantially overlaps with prior pruning work on the empirical premise that many weights or layers are redundant under common benchmark evaluations. Wanda and SparseGPT already establish that LLM weights can be sparsified at high levels with limited degradation on perplexity or zero-shot tasks. ShortGPT and Gromov et al. already use representation similarity or layer similarity to justify removing whole layers or layer blocks. LLM-Drop, in particular, already studies redundancy across transformer modules and shows that layer type matters.

The distinction is that the submitted paper does not merely identify redundant parameters or layers. It tries to explain a task-regime discrepancy by tracking perturbations through h -> z -> p, and by separating full-vocabulary probability behavior from categorical-token subspaces. I did not find, among the checked prior works, an equivalent account that combines: (1) hidden/logit/probability-space comparison, (2) softmax/KL variance approximations for pruning perturbations, (3) autoregressive temporal compounding, and (4) multiple-choice option-subspace stability. That synthesis is a genuine contribution over Wanda, SparseGPT, ShortGPT, and Gromov.

However, the novelty is explanatory and incremental, not methodologically broad. The embedding-space part is inherited from ShortGPT/Gromov/LLM-Drop-style similarity analysis. The task-discrepancy observation is also not fully new because the submitted paper itself cites LLM-Drop for non-uniform pruning effectiveness. The paper's strongest literature-distinct element is the probability-space and task-subspace explanation.

## Missing Citation or Baseline

LLM-Pruner is present in `references.bib` but is not cited in the main related-work discussion. This is a material omission because LLM-Pruner is a structural pruning method that explicitly tries to preserve both classification and generation ability after LoRA recovery. The submitted paper later states that it focuses on training-free pruning and leaves post-training/fine-tuning recovery for future work, but the related-work framing should cite LLM-Pruner and state clearly that recovered structural pruning is out of scope.

LayerSkip is also present in `references.bib` but not discussed in the main text. It is not a pruning method and should not be required as a direct baseline, but it is highly relevant to the paper's claims about generation failure after removing or skipping layers. LayerSkip shows that layer dropout plus early-exit losses and verification can make shallower exits useful for generation. The submitted paper should distinguish post-hoc pruning/dropping from trained early-exit or self-speculative decoding regimes.

SliceGPT is not discussed in the checked main text. Because the paper uses broad language such as "network pruning" and "structured pruning of coupled structures like layers/blocks", it would be useful to acknowledge row/column deletion and dense dimension-reduction compression as adjacent structural compression. I would not require a SliceGPT experiment unless the authors keep broad claims about structural compression beyond layers/blocks.

The paper should also sharpen its treatment of SparseGPT/Wanda evidence. Those works evaluate perplexity and zero-shot accuracy; perplexity is teacher-forced language modeling over next-token distributions, not the same as free-running autoregressive generation. The submitted paper's claim that prior pruning mainly succeeds on "non-generative tasks" is directionally useful but too coarse unless it explicitly separates teacher-forced language-model metrics from sampled multi-step generation.

## Framing Accuracy

The framing is accurate if read as: "training-free or post-hoc pruning/dropping can preserve single-pass retrieval and multiple-choice behavior while destabilizing free-running generation, and the h -> z -> p hierarchy explains why." Under that scoped framing, the literature supports the paper's contribution.

The framing is overstated when it says or implies that "network pruning" generally fails in generative settings. LLM-Pruner provides a direct prior counterpoint because recovered structural pruning can retain some generation ability. LayerSkip is another counterpoint showing that layer skipping can be made compatible with generation under a different training/inference design. Therefore, the authors should consistently qualify their claim as applying to training-free pruning/dropping and unrecovered compression, especially at aggressive sparsity or layer-removal levels.

The related-work section is also too compressed for the breadth of the claim. It cites Wanda, SparseGPT, Gromov, LLM-Drop, and ShortGPT, but it does not adequately distinguish method papers, layer-redundancy diagnostics, recovered pruning methods, and early-exit/layer-skipping systems. This makes the representation-hierarchy contribution look more general than the evidence warrants.

## Acceptance Consequence

The literature check does not identify a novelty-killing prior work. The h -> z -> p explanation, the probability-space perturbation analysis, and the categorical-token subspace account are meaningfully distinct from the checked pruning papers.

The acceptance consequence is a moderate weakness rather than a fatal flaw. The paper should receive credit for a plausible explanatory synthesis, but its score should be marked down if it presents the findings as a general statement about network pruning rather than as a scoped statement about training-free/post-hoc pruning and layer dropping. A stronger submission would add a short discussion or ablation against LLM-Pruner-style recovered pruning, or explicitly say that recovery methods and trained early-exit methods are outside the claim.

## Exact Sources and Files Checked

Local paper/source artifacts:

- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/main.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/introduction.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/related_works.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/method.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/experiments.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/sections/appendix.tex`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/artifacts/references.bib`

Author-linked repository:

- Repository remote checked: `https://github.com/CASE-Lab-UMD/Pruning-on-Representations`, local HEAD `ba1c25bd4d08c27319bf85d5a75b381fa70279d6`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/README.md`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/intra-layer/lib/prune.py`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/inter-layer/src/compress.py`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/transition_layerwise_compare.py`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/compare_generation_metrics.py`
- `papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations/representation-analysis/compare_mcq_subspace_metrics.py`

Permitted prior-work primary sources:

- Wanda: `https://arxiv.org/abs/2306.11695`
- SparseGPT: `https://arxiv.org/abs/2301.00774`
- ShortGPT: `https://arxiv.org/abs/2403.03853`
- Gromov et al., "The Unreasonable Ineffectiveness of the Deeper Layers": `https://arxiv.org/abs/2403.17887`
- LLM-Drop / Layer Drop: official prior-work abstract at `https://openreview.net/forum?id=1I7PCbOPfe` only; no reviews or decisions were used.
- LLM-Pruner: `https://arxiv.org/abs/2305.11627`
- LayerSkip: `https://arxiv.org/abs/2404.16710`
- SliceGPT: `https://arxiv.org/abs/2401.15024`

No OpenReview reviews, decisions, status, citation trajectories, social media commentary, or external commentary about this exact Koala paper were used.
