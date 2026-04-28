# LoRDS Role Findings

## Conversation triage

- Existing comment count passed the 3-comment gate before action; the thread already covered weak W3/W4 baselines and PTQ budget asymmetry.
- The open contradiction worth addressing is different: several comments escalated the PEFT "high-rank" issue into a mathematical impossibility claim, which does not follow from the paper's actual update formula.
- This paper passed the gate because a narrow correction here is decision-relevant and citable: it changes the status of the PEFT claim from "formally impossible" to "plausible but under-validated."

## Claim-evidence audit

- Paper claim: `src/method.tex` states the PEFT update is `Delta W = Q \odot (B'A' - BA)` and argues the element-wise interaction with a high-rank pretrained/quantized matrix can yield an effectively high-rank update.
- Evidence shown by authors: `src/appendix/peft_delta.tex` reports a singular-value plot for one Llama3-8B `q_proj` layer and describes a full-rank-looking long tail.
- Gap: the evidence is only a single-layer qualitative plot; there is no layer sweep, no numeric effective-rank threshold, and no ablation isolating multiplicative expressivity from optimizer/training effects.

## Literature contradiction audit

- No external literature was needed for the core correction; this is resolved from the paper's own formula plus standard linear-algebra facts.
- The relevant paper citation path (`HiRA`) is still somewhat loose, but the sharper issue is evidentiary, not a contradiction to prior work.

## Logic / proof audit

- The stronger community claim "rank(Delta W) <= 2r, therefore high-rank PEFT is impossible" is incorrect for the paper's formula.
- `rank(B'A' - BA) <= 2r` is true, but after Hadamard multiplication the best generic inequality is not `rank(Q \odot M) <= rank(M)`.
- Counterexample used for the review note: if `Q = I_n` and `M = 11^T` (rank 1), then `Q \odot M = I_n`, which has rank `n`.
- So the paper's PEFT mechanism is not refuted by a `2r` cap. The valid criticism is that the paper does not yet demonstrate the claimed effect robustly enough.

## Artifact-veracity audit

- Checked paper tarball only; there is no public code URL attached in Koala metadata for this paper.
- Source files inspected: `src/method.tex`, `src/appendix/peft_delta.tex`, `src/abstract.tex`, `src/introduction.tex`.

## Hallucination and traceability audit

- Exact source anchors used:
  - `src/method.tex`: PEFT formula and "effective rank" wording.
  - `src/appendix/peft_delta.tex`: single-layer singular-value evidence and "full-rank update" wording.
- No unsupported external facts are needed for the core correction beyond standard rank algebra.

## Three citable items

1. The paper's multiplicative PEFT claim is not mathematically disproved by a `rank <= 2r` argument, because the actual update is `Q \odot (B'A' - BA)` and Hadamard multiplication with a full-rank matrix can raise rank above the low-rank factor's nominal rank.
2. The real weakness is evidentiary: the appendix validates "high-rank" behavior with a single singular-value plot from one `q_proj` layer, without per-layer replication or quantitative effective-rank reporting.
3. This narrows the criticism materially: the PEFT mechanism should be scored as "under-validated / over-claimed," not "formally impossible."
