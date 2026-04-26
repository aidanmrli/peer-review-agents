# Role Findings

Paper: `a99e0983-dd14-4112-83ae-87fa04cdb5a0`

## Reproducibility lead

Central claim and reproduction target: verify that the released artifacts support the paper's claimed plug-and-play `PIPER` implementation, especially the MuJoCo-based Automated Dynamics Oracle, the auxiliary PINN, and the policy-regularized torque-control training setup used for the reported Fetch results.

Bottom line: I could not recover a runnable reproduction path from the released artifact. The paper bundle is source-only, and the method description depends on an apparently modified torque-control Fetch setup whose implementation details are not released.

## Reproducer A

Artifact-first check:

- Downloaded `https://koala.science/storage/tarballs/a99e0983-dd14-4112-83ae-87fa04cdb5a0.tar.gz`.
- Listed the bundle with `tar -tzf`.
- Contents are `preprint.tex`, `references.bib`, style files, `00README.json`, and static figures under `Images/`.
- I did not find code, MuJoCo XMLs, Gymnasium wrappers, training scripts, configs, checkpoints, logs, or evaluation outputs.

What was recovered:

- Enough to inspect the method and reported tables.
- Not enough to rebuild the experiments or verify the claimed plug-and-play integration.

## Reproducer B

Clean-room/specification check:

- Inspected `preprint.tex`.
- The paper explicitly assumes direct joint torque control (`preprint.tex:394-395`) and uses ADO queries plus a PINN inside the optimization loop (`preprint.tex:397-513`).
- The same paper evaluates `FetchReach-v4`, `FetchPush-v4`, `FetchSlide-v4`, and `FetchPickAndPlace-v4` (`preprint.tex:537-549`).

Clean-room conclusion:

- Reproducing the results requires the exact action-interface adaptation from standard Fetch tasks to the paper's torque-control formulation, plus the ADO/PINN implementation and hyperparameters.
- Those implementation details are not released in the artifact bundle, so the clean-room reproduction target is under-specified.

## Implementation auditor

Code/artifact/repo match:

- Koala metadata exposes no GitHub repository or code URL for this paper.
- The source bundle contains no executable implementation.
- The paper claims "no alterations to existing simulators or core RL algorithms" (`preprint.tex:55`, `87`, `549`), but it also introduces a `~162k`-parameter PINN and a runtime oracle path (`preprint.tex:408-410`, `486-513`, `549`, `711`).
- The most consequential missing piece is the environment/control wrapper: the text assumes torque outputs, while standard Gymnasium Fetch tasks are ordinarily discussed in end-effector control terms and the paper gives no released wrapper, XML patch, or environment-registration code.

Assessment:

- The artifact trail does not currently support the paper's implementation claim at the level needed for independent reproduction.

## Correctness specialist

Methods/metrics/conclusion risks:

- The published metric tables and conclusions rely on an unreleased training stack, so the reported gains cannot be independently verified from the supplied artifacts.
- The paper also defines stability as the standard deviation of success rate over the final 100 rollouts (`preprint.tex:553-559`) while reporting non-zero `sigma` values for 100% FetchReach success (`preprint.tex:577-584`), which makes the implementation details behind the reported metric even more important to inspect.

Score impact:

- Material reproducibility limitation for an empirical RL systems paper.

## Literature specialist

Novelty/framing against permitted prior work:

- The plug-and-play framing is potentially interesting precisely because it claims to avoid simulator/algorithm modifications.
- Without the actual wrapper/oracle/PINN code, it is hard to tell whether the contribution is a lightweight loss-term integration or a more substantial environment-specific engineering stack.
- That uncertainty weakens the evidence for the paper's practical contribution even if the underlying idea is promising.
