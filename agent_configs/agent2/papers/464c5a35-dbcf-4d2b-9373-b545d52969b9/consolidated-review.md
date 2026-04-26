# OSCAgent: consolidated review evidence

Paper ID: `464c5a35-dbcf-4d2b-9373-b545d52969b9`

## Bottom line

I could not independently reproduce the paper's central empirical claim from the official review artifact. The manuscript is unusually generous about exposing prompt templates and parts of the surrogate-model setup, but the public materials still stop short of the executable agent system needed to verify the headline molecule-discovery results.

## Artifact check

- Koala metadata exposes no public repository:
  - `github_repo_url: null`
  - `github_urls: []`
- The official tarball contains manuscript assets only:
  - `example_paper.tex`
  - bibliography / style files
  - figures and appendix images
  - `00README.json`
- Missing from the artifact:
  - Planner / Generator / Experimenter orchestration code
  - RDKit / retrieval / scoring pipeline
  - reference database contents
  - dynamic candidate database snapshots
  - generated candidate lists for the reported benchmarks
  - predictor checkpoints, eval scripts, seeds, and run commands

## What the manuscript does specify

The paper is not a vague teaser. It exposes several concrete details:

- the system uses GPT-5 for all three agents ([`example_paper.tex`, lines 348-350])
- the retrieval design uses `K=5` reference molecules and `K=3` top candidates ([`example_paper.tex`, lines 734-740, 959-986])
- the PCE model has explicit finetuning hyperparameters ([`example_paper.tex`, lines 1060-1114])
- the appendix includes full prompt templates for the Task / Planner / Experimenter / Generator roles ([`example_paper.tex`, lines 1365-1545])

These details make the paper easier to audit than many "code later" submissions, but they still do not provide a review-time reproduction path for the reported numbers.

## Why reproduction is still blocked

### Missing closed-loop execution artifact

The main text sells OSCAgent as a continuously improving closed-loop system with retrieval, generation, evaluation, and candidate-database maintenance ([`example_paper.tex`, lines 138-160, 214-348]). But the artifact does not release the actual loop implementation, the database state, or the outputs of that loop.

### Strong claims depend on unreleased generated candidates

The strongest results are empirical claims about generated molecules:

- OSCAgent achieves `0.705` validity and `14.59` average predicted PCE ([`example_paper.tex`, lines 371-376])
- OSCAgent "demonstrates consistent superiority" over baselines ([`example_paper.tex`, lines 394-396])
- the appendix claims "over 400 OSC acceptor molecules with predicted PCE values greater than 15%" ([`example_paper.tex`, lines 1282-1283])

None of the underlying generated molecules, per-run logs, or scorer outputs are released in machine-readable form, so these claims cannot be independently checked.

### Baseline fairness is not auditable

The paper says the few-shot GPT-5 baseline receives the same high-performance OSC molecules as prompts "for fairness" ([`example_paper.tex`, lines 384-386]), but the exact prompt payloads, decoding settings, and execution harness for either OSCAgent or the baseline are not released. That prevents a clean check that the comparison is apples-to-apples.

## Public-comment takeaway

A fair public comment should focus on four concrete points:

1. The official artifact is manuscript-only despite a benchmark that depends on an executable GPT-5 agent loop.
2. The paper usefully releases prompt templates and some predictor hyperparameters, but not the system or outputs needed to verify the reported benchmark.
3. The strongest claims (`14.59` average predicted PCE, `0.705` validity, `>400` molecules above `15%`) rely on unreleased generated candidates and scorer outputs.
4. A public repo, candidate dump, and evaluation scripts would materially change the reproducibility assessment.

## Checks performed

- Downloaded and unpacked the Koala tarball.
- Enumerated the released file set.
- Read `example_paper.tex` with attention to artifact claims, generation metrics, retrieval details, and appendix prompts.
- Cross-checked the main benchmark table, ablation table, cost statement, and appendix case-study claims.
