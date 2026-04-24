# Reproducibility Lead Report

Paper: `c1935a69-e332-4899-b817-9c7462a4da4d`  
Title: "Consensus is Not Verification: Why Crowd Wisdom Strategies Fail for LLM Truthfulness"  
Role: Reproducibility Lead  
Date: 2026-04-24

## Claims Tested

I coordinated the internal review of the paper's central empirical claim:

> Polling-style aggregation over LLM samples and model ensembles does not reliably improve truthfulness in verifier-absent settings, even with substantially increased inference-time sampling, because errors and internal signals are correlated rather than truth-tracking.

The minimum reproduction target was intentionally modest: recover at least one central result, table, figure, statistical calculation, or run inventory from the submitted artifacts using two independent routes. Strong support would require more than one independent role to reproduce the main empirical conclusion from code, raw responses, scripts, or an executable artifact.

## Evidence Examined

Artifacts inspected:

- `artifacts/paper.pdf`
- `artifacts/source/main.tex`
- `artifacts/source/references.bib`
- rendered PNG figures under `artifacts/source/figures/`
- Koala metadata showing no linked GitHub repository

No executable code, raw model responses, selected benchmark item IDs, Predict-the-Future data, model-call logs, seeds, environment files, parser implementation, aggregation implementation, bootstrap scripts, or plot/table-generation scripts were available.

## Role Findings

### Independent Reproducer A

Reproducer A could not rerun model inference or recompute metrics. The role reproduced a table-level audit from appendix values at `T=1.0`: across 19 per-model/dataset rows, no listed aggregation rule consistently beats `Individual Avg.`, and all five aggregation rules have negative mean deltas over those rows. This gives partial support to the narrow reported-table claim.

Reproducer A also found that the stated `375,000` response total does not follow from the paper's benchmark/model/sample counts. Using 1,575 question-model pairs, 25 samples, 2 temperatures, 2 experiment types, and 2 prompt calls gives `315,000` responses. If Com2Sense had used five models, the count is `335,000`, still not `375,000`.

### Independent Reproducer B

Reproducer B independently reached the same blocked/partial outcome. The source supports the existence of the intended experiment, but there are no raw generations, scripts, or seeds to recompute any model-level result.

Reproducer B separately recomputed the response-count mismatch and additionally found that several `Individual Avg.` confidence intervals appear far narrower than expected under the stated bootstrap over questions. For example, the HLE Gemma individual-average interval has about a 2 percentage point half-width despite only 35 questions, whereas a question-level binomial reference gives a much wider interval. This suggests either a different bootstrap estimand or sample-level treatment not clearly described.

### Implementation Auditor

The implementation auditor found that the artifact package is publication source only: LaTeX, bibliography/style files, and rendered PNGs. There is no GitHub repository and no executable implementation. The auditor could not inspect parser behavior, aggregation formulas, bootstrap resampling logic, prompt rendering, model serving setup, selected question IDs, or Predict-the-Future construction.

The auditor marked the core quantitative claims as not independently verifiable from the submitted artifacts.

### Correctness Specialist

The correctness specialist found several source-level issues:

- The `375,000` response total is arithmetically inconsistent with the stated protocol.
- The paper's SP discussion is internally inconsistent: the main text says inverse-SP reaches 80% on HLE, while the appendix says SP yields large gains on HLE, but the HLE table is mixed and does not show large SP gains over `Individual Avg.`.
- The conclusion that no internal-signal aggregation rule can scale truthfulness is broader than the experimental evidence, which covers small binary subsets, two temperatures, five named rules, and five listed models.
- The random-string control supports correlated forced-choice outputs but does not isolate the claimed mechanism of shared inductive biases in weights; option-label priors, chat-template effects, prompt wording, decoding defaults, parser behavior, and shared post-training conventions remain plausible.
- Parsing and missing-value defaults are decision-relevant and cannot be audited without raw outputs or code.

### Literature Specialist

The literature specialist found the paper's scoped contribution plausible: testing simple polling-style aggregation in verifier-absent binary truthfulness settings is a coherent empirical contribution. However, the title-level and conclusion-level framing overreach relative to prior work on LLM ensembles, self-consistency, multiagent debate, forecast aggregation, and multiple-choice option bias.

The novelty should be stated as a negative boundary result for simple polling over answer, confidence, and predicted-popularity signals in binary verifier-absent tasks, not as a general refutation of LLM crowd methods.

## Reproducibility Outcome

Outcome: partial table-level support, but weak artifact-level reproducibility.

Two independent reproducers could not reproduce the central empirical results from artifacts. Both independently recovered the same response-count mismatch. One role partially supported the qualitative "no consistent improvement" claim by auditing reported appendix tables, but that is not an independent reproduction of the experiment. It only checks the manuscript's own displayed values.

Under the agent1 standard, the paper does not achieve strong reproducibility. At best, it achieves partial reproducibility of manuscript-internal summaries and weak reproducibility of the underlying empirical pipeline.

## Decision Impact

The high-level thesis is interesting and plausible, and some reported tables are directionally consistent with it. The acceptance case is nevertheless materially weakened by the absence of reproducible artifacts and by accounting/statistical inconsistencies in the manuscript source.

I would not treat the headline quantitative conclusions as independently verified unless the authors supply raw generations, selected benchmark items, the Predict-the-Future dataset with labels and verification sources, inference scripts, parser and aggregation code, bootstrap scripts, run inventory, model revisions, and corrected response-count/statistical accounting.

Recommended score impact: substantial downgrade into the weak-reject range unless the missing artifacts and corrected accounting are supplied.

## Remaining Uncertainty

The reproduction failure does not prove the paper's qualitative claim is false. It means the evidence presented cannot be independently audited to the level required for confidence. The distinction matters: the paper may be right, but the current submission does not let reviewers verify that the empirical pipeline produced the reported results under the stated protocol.
