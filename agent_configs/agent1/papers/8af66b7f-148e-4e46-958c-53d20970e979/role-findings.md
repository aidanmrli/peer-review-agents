Paper: `8af66b7f-148e-4e46-958c-53d20970e979`
Title: `Efficient Multi-round LLM Inference over Disaggregated Serving`
Date: `2026-04-28`

## Central claim and reproduction target

The paper claims its offline deployment planner is an Integer Linear Programming / MILP formulation that can be solved efficiently and yields the optimal prefill/decode deployment configuration.

## Paper and artifact evidence checked

- Downloaded and extracted the Koala tarball for `8af66b7f-148e-4e46-958c-53d20970e979`.
- Inspected `tmp/8af66/example_paper.tex` directly.
- Relevant source locations:
  - Planner setup and Eq. (deployment formulation): lines 536-557.
  - Implementation claim that SCIP solves the same formulation: lines 572-573.

## Reproducibility result from the smallest meaningful check

I could not reconstruct an executable MILP from the paper's own mathematical statement. The issue is not missing code alone; the published formulation itself is under-specified as a linear program.

- Lines 539-540 introduce only integer replica-count variables `x^(n)` and `y^(n)`.
- Lines 549-550 then impose constraints `Z >= tau_pre(n)` / `Z >= tau_dec(n)` only for `n` "where `x^(n) >= 1`" or "where `y^(n) >= 1`".
- That condition is a logical implication depending on the decision variables. As written, it is not a standard linear constraint set. A MILP needs explicit activation binaries, indicator constraints, or a big-M reformulation to encode "worker type n is instantiated".
- Lines 556-557 nevertheless state that a MILP solver explores the feasible region defined by the integer vectors `x` and `y`, with no additional binaries or reformulation described.

## Implementation or correctness risks

- The planner is not reproducible from the manuscript because multiple inequivalent encodings are possible once activation variables are introduced.
- The claimed global-optimum guarantee depends on the exact encoding; without the missing linearization, the guarantee is not auditable.
- If the actual code uses a different formulation than the paper, reviewers cannot tell whether the reported planning results correspond to the published objective.

## Novelty/framing context

This is narrower than the existing thread's code-availability complaints. Even if the implementation were released later, the current paper still does not specify a complete MILP.

## Decision impact

This is a material reproducibility gap for a systems paper whose deployment-planning claim is one of the stated technical contributions. It does not by itself prove the planner is wrong, but it means the planning component is not currently implementable from the paper alone.
