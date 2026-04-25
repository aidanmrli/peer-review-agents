# Expert Threshold Routing - Correctness Specialist

- Paper ID: `acca775c-254b-410c-9252-c37ed998431f`
- Title: `Expert Threshold Routing for Autoregressive Language Modeling with Dynamic Computation Allocation and Load Balancing`
- Assigned role: Correctness Specialist
- Date: 2026-04-25

## Task scope

Check the paper's definitions, experiment logic, metrics, and conclusions for decision-relevant technical errors or unsupported claims.

## Candidate error 1: paper-code inconsistency in MoE configuration

- Location: `artifacts/v2.tex:304-309`, `artifacts/v2.tex:715-761`
- Code evidence: `configs/mlp/et.yaml:9-10`, `configs/mlp/ec.yaml:9-10`, `src/models/model_base.py:123-130`, `src/models/engines/common.py:13-21`

The paper describes `G=1,E=16` with 16 routed experts plus one shared expert. The repository configs use `G=2,E=8`; the code asserts that shared experts require `G>=2`. The per-expert target formula for shared experts is `n_tokens * (g - 1) // (g * e)`, so the stated `G=1,E=16` would imply zero routed tokens under the released code.

Severity: major.

Consequence: The exact reported architecture and routing target cannot be reconstructed from the released implementation. This affects active parameter accounting, routed expert width, capacity targets, and the fairness of comparisons to TC/EC baselines.

## Candidate error 2: "1.6x fewer tokens" is not verifiable from released data

- Location: `artifacts/v2.tex:138`, `artifacts/v2.tex:146-148`
- Evidence: no raw loss curve table/CSV/JSON found in artifacts or repo; only static figure PDFs/PNGs are provided.

The claim may be derived from the d20 loss curve, but the release does not provide the underlying loss-versus-token points or interpolation rule. A static plotted figure is not enough to verify a numeric 1.6x token-efficiency claim.

Severity: major for reproducibility, moderate for correctness.

Consequence: The CE gap in Table d20 can be read from the paper, but the token-efficiency equivalence cannot be independently checked.

## Candidate error 3: d20 baseline coverage is narrower than the main framing suggests

- Location: `artifacts/v2.tex:312-318`, `artifacts/v2.tex:349-363`

The main text says TC variants include no load balancing, auxiliary loss, and loss-free load balancing (`artifacts/v2.tex:315`), but the d20 table reports only `TC aux` against EC and ET (`artifacts/v2.tex:359-362`). The broad abstract and introduction compare ET to "TC-MoE" at the headline scale, but the strongest d20 claim rests on one TC baseline variant.

Severity: moderate.

Consequence: The headline d20 delta is less robust than the wording suggests. The d12 table has a more complete TC suite; the d20 headline should be framed as comparison to the reported TC-aux d20 run unless additional d20 TC variants are supplied.

## Candidate error 4: load-balancing conclusion depends on unreleased diagnostics

- Location: `artifacts/v2.tex:947-955`

The paper states capacity constraints are triggered infrequently and train-inference mismatch is minimal. The mechanism itself is plausible, but the claim depends on raw expert-usage, saturation, and starvation logs after warmup. Only a static figure is released; no logs are available for independent checking.

Severity: moderate.

Consequence: This is an unsupported diagnostic rather than a fatal methodological flaw. It still matters because ET load balance is in expectation and training-time capacity clamping is explicitly absent at inference.

## Candidate error 5: CORE metric description appears underspecified

- Location: `artifacts/v2.tex:857-870`
- Code evidence: `eval_core.py:57-60` requires a checkpoint; no checkpoints are released.

The paper describes centered accuracy and final mean score, but does not release per-task outputs or checkpoints. This prevents recomputation of CORE values in Tables 1 and 2.

Severity: moderate.

Consequence: The CORE gain is not independently verifiable.

## Confidence level

High for candidate errors 1 and 2. Moderate for errors 3-5 because they are mainly overclaiming/underspecification rather than mathematical contradictions.

## Decision impact

The paper has a plausible technical idea but overstates empirical certainty. The configuration inconsistency and missing raw evidence materially reduce confidence in the reported performance claims.
