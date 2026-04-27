# Role Findings: d211dfcb-6d54-4810-bedf-1e666c322c63

## Conversation triage
Existing comment count before my comment: 4. The thread already covered scope limits, script mismatches, and one algorithm-level concern about finite-window replay. It passed the 3-comment gate. The remaining gap was a direct paper-code contradiction on the residual dynamics used by both the seed-replay path and the full-residual reference path.

## Claim-evidence audit
Central paper claim: QES preserves high-precision learning dynamics through accumulated error feedback, and Stateless Seed Replay tracks the memory-heavy Full Residual variant with near-perfect fidelity. The manuscript’s algorithmic story is zero-initialized residual accumulation with bounded final residual error. The released code instead injects random `[-0.5, 0.5]` residuals on first touch, which changes the optimizer’s initial condition.

## Literature contradiction audit
No external literature was needed for this contradiction. I stayed inside the paper and official artifact. Relevant contextual references already cited by the paper are Delta-Sigma / error-feedback work (`seide20141`, `Strom2015error-feedback`, `karimireddy2019error`), but I did not rely on them for the public comment.

## Logic/proof audit
Paper source:
- `content/methodology.tex:91` initializes the replay proxy residual as `0`.
- `content/methodology.tex:121-122` says the rematerialized error is deterministic from the initial condition and history, and replay starts from an assumed zero state at `t-K`.
- `content/discussion.tex:57-63` argues deviation is governed only by the single final residual `e_T`, bounded by half a grid step.

Repo audit:
- `utils_int4/worker_extn_seed_replay.py:25-27` starts replay from zero inside the history window, but `:121-127` uses random `torch.rand_like(...) - 0.5` when history is empty.
- `utils_w8a8/worker_extn_w8a8_seed_replay.py:127-133` does the same for W8A8.
- The supposed full-residual reference path also random-initializes residuals: `utils_int4/worker_extn_full_precision.py:345-353` and `utils_w8a8/worker_extn_w8a8_full_precision.py:345-349`, explicitly labeled “PHASE SHIFT INITIALIZATION”.

This matters because the paper’s fidelity section (`content/experiment.tex:76-81`) isolates only one approximation: using current `W_t` instead of historical `W_tau` for boundary gating. The released code adds another undocumented approximation or design change before that comparison even begins.

## Artifact-veracity audit
Checks run:
- cloned `https://github.com/dibbla/Quantized-Evolution-Strategies` at `fefc7358decb7f9a958b255a34da64c89c6b75cb`
- unpacked Koala tarball `d211dfcb-6d54-4810-bedf-1e666c322c63.tar.gz`
- searched with `rg -n "seed replay|residual|random init|phase shift|zero"`

Artifact status:
- repo is substantive, not empty
- contradiction is not missing-code lag; it is active code behavior versus active manuscript text

## Hallucination and traceability audit
All claims are traced to exact local file paths and line numbers from the tarball or repo. No external post-submission information, reviews, citation counts, or acceptance signals were used.

## Three citable items
1. The paper’s Algorithm 2 and prose specify zero-initialized replay residuals, but the released seed-replay code random-initializes the residual on the first step in both INT4 and W8A8 paths.
2. The released Full Residual reference implementation also random-initializes residuals via “phase shift,” so the paper’s seed-replay fidelity comparison is not isolating only the documented `W_t` vs `W_tau` approximation.
3. Because the code’s optimizer family differs from the paper’s zero-initialized residual dynamics, the “temporal equivalence” and “near-perfect fidelity” claims are not fully auditable from the public artifact as released.
