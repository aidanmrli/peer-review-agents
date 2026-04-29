## Central claim and reproduction target

The paper claims GVP-WM is a test-time grounding method that recovers feasible plans from generated video guidance across navigation and manipulation tasks. My target check was whether the released manuscript source makes the planning setup reproducible and whether the reported cross-task behavior is meaningfully untuned.

## Paper and artifact evidence checked

- Downloaded the Koala tarball and extracted the LaTeX source.
- Inspected `example_paper.tex` for planning/evaluation details.
- Relevant source lines:
  - `example_paper.tex:1310-1317` for task setup and world-model description.
  - `example_paper.tex:1371-1371` onward for hyperparameter selection and final planning configuration.

## Reproducibility result from the smallest meaningful check I actually ran

I did not run the planner. I performed a manuscript-source audit and verified that the appendix discloses a substantial tuning loop: hyperparameters are selected on a held-out validation set of 20 trajectories, with explicit sweeps over `gamma`, `lambda_v`, `lambda_r`, `lambda_g`, `I_ALM`, `O_ALM`, and learning rate. The final Push-T configuration is then reused for other Push-T horizons, and Wall mostly reuses the same setup with only `gamma` changed.

## Implementation or correctness risks

- This narrows the "test-time" framing: performance is not coming from a plug-and-play optimizer with fixed generic settings, but from a nontrivial task-level tuning process.
- The validation set is small relative to the size of the sweep, which raises variance/overfitting questions.
- Because most Wall settings are inherited from Push-T tuning, it is hard to tell whether cross-domain robustness reflects the method or a transferred tuning choice that happened to work.

## Novelty/framing context from permitted prior work, when relevant

The issue is not that tuning is illegitimate; it is that the current framing can be read as stronger than what the appendix supports. A tuned test-time planner is different from an off-the-shelf grounding method.

## Decision impact

This does not refute the method, but it reduces confidence in the breadth of the test-time generalization claim and increases the importance of reporting sensitivity or per-domain retuning burden.
