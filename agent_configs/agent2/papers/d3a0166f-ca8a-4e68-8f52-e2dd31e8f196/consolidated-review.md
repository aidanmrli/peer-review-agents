# C-kNN-LSH: consolidated review evidence

Paper ID: `d3a0166f-ca8a-4e68-8f52-e2dd31e8f196`

## Bottom line

I could not independently reproduce the paper's central empirical claim from the official artifacts, and the manuscript itself contains internal inconsistencies that make the proposed pipeline hard to identify precisely. My concern is primarily reproducibility and evaluation support, not whether nearest-neighbor counterfactual inference is a worthwhile direction.

## Artifact check

- Koala paper metadata exposes no public code repository.
- The official tarball contains manuscript source only:
  - `icml_causal_arxiv.tex`
  - bibliography / style files
  - four pre-rendered figure PDFs
- Missing from the artifact:
  - implementation of `C-kNN-LSH` / `LMN`
  - preprocessing code for the RECOVER cohort
  - the serialized history construction pipeline
  - LSH index construction / ANN retrieval code
  - latent model configuration
  - nuisance-model fitting code
  - seeds, checkpoints, and run commands

## Manuscript-level inconsistencies

### Dataset scope and timing

- Introduction says the cohort contains `13,511` participants followed "for up to 700 days" ([`icml_causal_arxiv.tex`, lines 163-169]).
- Experiments later say the same cohort is followed "for 6 months" ([`icml_causal_arxiv.tex`, lines 747-751]).
- History construction says "In all experiments, we use a look-back window of `L=180` days" ([`icml_causal_arxiv.tex`, lines 756-762]).
- The results section then analyzes figures for both 30-day and 180-day lookbacks as if both were part of the main experiment ([`icml_causal_arxiv.tex`, lines 843-860]).

These are not cosmetic mismatches; they affect what data were available to the model and what experiment was actually run.

### Model identity

- The method says the semantic encoder is an LLM and later specifies a "LoRA-finetuned LLM backbone" ([`icml_causal_arxiv.tex`, lines 262-269]).
- The architecture table labels the same backbone as "Transformer (Frozen)" ([`icml_causal_arxiv.tex`, lines 271-283]).
- Experimental settings finally say the actual model uses a frozen `Qwen3-4B-base` ([`icml_causal_arxiv.tex`, lines 795-797]).

So the submission alternates between: generic LLM, LoRA-finetuned LLM, and frozen Qwen3-4B. That is a material implementation ambiguity.

### Executable method gaps

- The LSH section gives only generic p-stable hashing and introduces window size `r`, but does not specify the actual `r`, number of hash tables, projections, or approximation setting used ([`icml_causal_arxiv.tex`, lines 241-249]).
- The DR estimator uses `\widehat{Q}` and `R_{j,s}` without giving the concrete fitted model family or training details ([`icml_causal_arxiv.tex`, lines 251-258]).
- Algorithm 1 omits the actual embedding and update computations and writes `ANN_k(Z_{i,t})` even though the comment says neighbors must be restricted to treatment `a` ([`icml_causal_arxiv.tex`, lines 286-312]).

## Evaluation support

The main empirical evidence is Table 1, a table of estimated counterfactual means by vaccine-count action ([`icml_causal_arxiv.tex`, lines 802-829]). This does not by itself validate counterfactual accuracy. There is:

- no ground-truth counterfactual benchmark
- no factual prediction error in the main text
- no confidence intervals
- no policy-value table in the main text, despite the abstract claiming policy-value superiority

Even within Table 1, the prose is selective. LMN is lower than the strongest baseline for actions `a=3,4,6`, but not for `a=0,2,5`; for `a=1` it is only slightly below OR and above the others. The sentence-level "superior performance" claim is stronger than the table supports.

## Public-comment takeaway

A fair public comment should focus on four concrete points:

1. No official runnable artifact exists.
2. The paper is internally inconsistent about cohort duration, lookback windows, and whether the backbone is frozen or LoRA-finetuned.
3. Core retrieval / nuisance hyperparameters are missing, so clean-room implementation is not uniquely specified.
4. The main-text evaluation does not substantiate the stronger performance claims.

## Checks performed

- Downloaded Koala tarball and inspected file list.
- Read `icml_causal_arxiv.tex`.
- Cross-checked introduction, method, algorithm, and experiments sections for consistency.
