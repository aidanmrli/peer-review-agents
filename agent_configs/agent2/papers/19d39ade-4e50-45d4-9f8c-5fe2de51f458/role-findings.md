## Reproducibility lead: central claim and reproduction target

Central claim: fixed-weight test-time search over a DISCO-learned operator dictionary can outperform direct prediction and gradient-based adaptation on zero-shot PDE parameter extrapolation and unseen operator compositions, while also recovering PDE coefficients. Reproduction target: rebuild the custom DISCO pretraining pipeline, extract the operator dictionary used at test time, rerun beam/uniform search, and recover Tables 1-2 plus the parameter-identification plots.

## Reproducer A: artifact-first check

I downloaded the Koala PDF and tarball into `papers/19d39ade-4e50-45d4-9f8c-5fe2de51f458/`. The tarball contains only LaTeX sources, figures, and bibliography (`source/icml2026.tex`, `plots/*`, style files). Koala lists no `github_repo_url` and no `github_urls`.

Artifact-first conclusion: there is no runnable code, config directory, checkpoint, or dataset-generation script for the modified DISCO model, the operator dictionary extraction, or the beam/uniform search procedure.

## Reproducer B: clean-room/specification check

The text is stronger than a typical paper-only release: it gives the PDEs, parameter ranges, rollout horizons, search hyperparameters, DISCO architectural changes, and pseudocode for beam search and random search. However, several decision-critical details remain underspecified:

- The method relies on a dictionary built from `f_i = psi_alpha(u_i^{1:L})` for training trajectories (`icml2026.tex:299-301`), but the paper does not state the resulting dictionary size per benchmark, whether every training trajectory contributes an operator, whether near-duplicate operators are deduplicated, or how the beam-search subsamples of `N=256/96/40/17` operators are chosen (`icml2026.tex:431-433`).
- The custom DISCO recipe changes the original method through a bottleneck layer and trajectory-pair / environment-codebook training (`icml2026.tex:720-746`), but still omits some implementation details needed for exact recovery, such as random-seed handling, validation-selection policy, and the codebook update specifics beyond a high-level EMA description.
- For Navier-Stokes, the paper specifies viscosities and grid sizes, but not the number of Euler and diffusion training trajectories or the exact split sizes used to produce the final dictionary.

Clean-room conclusion: the paper is reproducibility-aware, but not enough to uniquely reconstruct the operator library and search space that drive the headline zero-shot results.

## Implementation auditor: code/artifact/repo match

The strongest claims in Tables 1-2 and in the scaling analysis depend on a nontrivial interaction between:

1. modified DISCO pretraining,
2. extraction of a large operator dictionary from training trajectories,
3. benchmark-specific subsampling of that dictionary, and
4. beam search with operator splitting.

Because none of this pipeline is released, an auditor cannot verify whether the gains come from the proposed search strategy itself or from unreported implementation choices in dictionary construction.

## Correctness specialist: methods, metrics, or conclusion risks

The method is plausible and the paper does provide beam-search pseudocode plus dataset formulas. My main risk is not obvious mathematical inconsistency; it is hidden experimental sensitivity. The reported parameter-identification claim also depends on tracing selected operators back to training trajectories with known coefficients (`icml2026.tex:505-509`), which is impossible to audit without the actual dictionary and selection code.

## Literature specialist: novelty/framing against permitted prior work

The framing is coherent: this is a search-based fixed-weight alternative to fine-tuning and in-context PDE adaptation. My reservation is calibration. The paper reads like a reproducible systems contribution, but the release quality is still below that bar because the new contribution is exactly the unreleased operator-search pipeline.
