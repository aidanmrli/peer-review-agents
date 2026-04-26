## Reproducibility lead
Central claim and reproduction target: reproduce the sequential counterfactual estimator `C-kNN-LSH` / `LMN` on the NIH RECOVER Long COVID cohort and verify the claimed superiority in policy-value / counterfactual estimation.

Result: blocked before execution. The official artifact is manuscript source only (`icml_causal_arxiv.tex`, bibliography, figures). There is no code, config, environment, dataset loader, preprocessing script, checkpoint, or command line.

## Reproducer A
Artifact-first check.

- Koala metadata shows `github_repo_url: null` and `github_urls: []`.
- Local tarball contents are LaTeX plus four PDF figures; no implementation.
- The paper claims a fairly specific stack: LLM encoder, variational bottleneck, LSH retrieval, DR correction, and Long COVID preprocessing, but the artifact does not expose any of it.

Decision impact: no runnable reproduction path exists from the official materials.

## Reproducer B
Clean-room/specification check.

- The method section defines LSH only at a generic level; key executable choices are missing: latent dimension, `k`, number of hash tables, number of projections, window size `r`, ANN library, optimizer, learning rate, epochs, batch size, LoRA rank, and decoder / outcome-head architecture.
- Equation (db) is underspecified: it uses `\widehat{Q}` and `R_{j,s}` without fully defining the local model form or fitting procedure.
- Algorithm 1 omits the actual embedding and loss-computation steps and writes `ANN_k(Z_{i,t})` while the comment says neighbors must be treated with action `a`, so the retrieval rule is not operational as written.

Decision impact: even a careful clean-room pass cannot recover a unique implementation.

## Implementation auditor
Code/artifact/repo match.

- Method text says the backbone is an LLM semantic encoder, first generically in Sec. 2.2 and then specifically as a LoRA-finetuned model in Sec. 2.6.
- Experimental settings later say the actual model is a frozen `Qwen3-4B-base`.
- Table 1 and the algorithm refer to `LMN`, while the paper title and method sell `C-kNN-LSH`; the naming suggests the submitted system is a composite pipeline, but the decomposition is not documented as a reproducible package.

Decision impact: the described system changes identity across sections and is not auditable.

## Correctness specialist
Methods / metrics / conclusions risks.

- The paper claims superiority, but Table 1 is just one table of estimated counterfactual means. There is no ground-truth counterfactual benchmark, no factual prediction error, no confidence intervals, and no policy-value table in the main text.
- Within Table 1, LMN is not uniformly better: for actions `a=0,2,5`, at least one baseline yields lower severity than LMN; the prose only highlights the subset `a=3,4,6`.
- The dataset description is internally inconsistent: introduction says the cohort is followed for up to 700 days, experiments say 6 months, and the lookback section says all experiments use 180 days while later figures compare 30-day and 180-day lookbacks.

Decision impact: the empirical section does not support the abstract-level performance claim at current specificity.

## Literature specialist
Novelty / framing against prior work.

- The paper frames the method as an LLM-augmented causal estimator for text-like histories, but the actual histories are serialized wearable summaries and clinical covariates.
- The novelty appears to be a synthesis of known ingredients: VAE compression, ANN / LSH retrieval, and doubly robust estimation. That can still be useful, but the manuscript needs a clearer boundary between generic components and the specific new contribution.

Decision impact: novelty is plausible as a systems combination, but the current framing overstates what is shown.
