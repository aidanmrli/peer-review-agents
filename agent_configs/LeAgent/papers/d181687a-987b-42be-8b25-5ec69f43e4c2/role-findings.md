# R2-Router Role Findings

## Conversation triage

- The paper already passed the 3-comment gate many times over; the active thread is about whether the length-compliance concern is resolved or merely mitigated.
- Existing comments identified the requested-budget versus actual-token issue, but the paper source itself exposes a sharper internal contradiction with exact line-level anchors.
- This is decision-relevant because the theorem, routing objective, and 4-5x cost claim all depend on what "cost" actually means.

## Claim-evidence audit

- Main claim: R2-Router jointly selects an LLM and token budget, and achieves comparable quality at 4-5x lower cost.
- Evidence path in the source is internally mixed:
  - `main.tex:407-409` defines the optimization target with `C(b)` equal to `budget * per-token cost`.
  - `main.tex:595-597` says responses are generated under budget prompts, "enforced by truncation," and annotated with actual token counts.
  - `main.tex:766` then says that when models exceed budget, "the observed cost is higher than intended" and the learned curve reflects that reality.
- These statements do not specify a single cost variable for training/evaluation.

## Literature contradiction audit

- No external literature was required for this comment. The contradiction is internal to the paper's formalism and data pipeline.

## Logic / proof audit

- The optimization theorem and routing equations use `C(b)` as a function of the requested budget alone.
- But the appendix defense of low compliance assumes the router learns from realized response costs, which may exceed `C(b)`.
- If training/evaluation truly use actual token costs, then the formal objective and theorem are mis-specified.
- If training/evaluation instead use requested budgets or truncation caps, then Appendix A overstates the mitigation because over-budget generations are not being represented as higher realized costs in the same way the appendix claims.

## Artifact-veracity audit

- Checked only the Koala tarball.
- Relevant source anchors:
  - `main.tex:407-409`
  - `main.tex:561`
  - `main.tex:595-597`
  - `main.tex:758-766`

## Hallucination and traceability audit

- All claims in the planned comment come directly from the manuscript source.
- No external code, API, or repo claims are used.

## Three citable items

1. The paper's formal objective defines cost as requested budget times per-token price (`C(b)`), but Appendix A defends the method using realized over-budget costs, so the optimization target and the compliance mitigation are not written in the same variable.
2. The dataset section simultaneously says budgets are "enforced by truncation" and that actual token counts are recorded, but the evaluation section never states whether deferral curves use requested budgets, truncated caps, or actual token costs.
3. Because the 4-5x savings claim and Theorem 4.3 rely on the meaning of cost, the paper needs an explicit accounting rule before the compliance objection can be considered closed.
