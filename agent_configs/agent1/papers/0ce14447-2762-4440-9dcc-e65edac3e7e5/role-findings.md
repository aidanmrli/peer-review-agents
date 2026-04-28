# Sign Lock-In: role findings

## Central claim and reproduction target
- Central claim: the paper identifies sign lock-in as the reason learned signs remain effectively incompressible in the sub-bit regime, and theory-guided interventions make sub-bit compression practical.
- Reproduction target for this cycle: verify what the paper's strongest practical compression result actually implements, and whether it is natural sign-lock-in preservation or a stricter constrained-training setup.

## Paper and artifact evidence checked
- Fetched and unpacked the Koala source tarball for `0ce14447-2762-4440-9dcc-e65edac3e7e5`.
- Reviewed the main text intervention section around the sign-template discussion and the appendix section for the "zero-cost sign template" compression experiment.
- Read the current Koala discussion to avoid repeating the optimizer-assumption thread and to place the contribution as an implementation-scope clarification.

## Reproducibility result from the smallest meaningful check
- The strongest sub-bit result is not a pure "natural lock-in" result. The appendix explicitly says the reported compression setup applies to a fixed subset of targeted linear tensors while all other parameters remain full precision.
- The appendix also states that after each optimizer update, the method applies an element-wise hard projection to enforce `sign(W)=T` exactly on all targeted layers.
- Therefore the practical sub-bit result is a constrained-training plus selected-layer-compression result, not merely evidence that ordinary sign persistence alone yields deployable whole-model sub-bit compression.

## Implementation or correctness risks
- Main-text framing emphasizes minimal interventions, but the appendix's strongest compression path depends on exact hard projection after every optimizer step.
- The paper reports effective bits-per-weight for the targeted matrices, while the untouched parameters remain full precision; this weakens any whole-model reading of "surpassing the one-bit wall."
- This is not a fatal flaw for the empirical sign-lock-in phenomenon itself. It is a scope/framing risk for the practical compression claim.

## Novelty/framing context
- The sign-lock-in observation and stopping-time framing appear novel and useful.
- The practical compression contribution is narrower than the top-line framing suggests because it relies on explicit sign fixing and partial-model targeting, not only passive lock-in.

## Decision impact
- Positive update: the paper has a real mechanism for making selected-layer sign storage effectively zero.
- Negative update: the current manuscript over-compresses the distinction between observing sign lock-in and enforcing sign templates during training, so I would discount the practical "sub-bit deployment" strength unless end-to-end whole-model accounting is shown.
