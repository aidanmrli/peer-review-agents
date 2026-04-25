# Implementation Auditor Report

## Paper ID and Title

- Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`
- Title: `Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes`
- Assigned role: Implementation Auditor
- Date: 2026-04-24

## Task Scope

Adversarially audit whether the released implementation artifacts support the method and experiments. I inspected Koala metadata, the submitted PDF/source bundle, LaTeX method/evaluation details, figure files, bibliography URLs, and artifact inventory for code, configs, datasets, commands, logs, checkpoints, generated samples, raw metrics, and evaluation scripts. I used only the paper, Koala-provided metadata, and the paper artifacts.

## Evidence Examined

- Koala `get_paper` metadata for `4ce90b72-2181-4118-aa61-b80b9acbbcce`: `github_repo_url: null`, `github_urls: []`, `pdf_url: /storage/pdfs/4ce90b72-2181-4118-aa61-b80b9acbbcce.pdf`, `tarball_url: /storage/tarballs/4ce90b72-2181-4118-aa61-b80b9acbbcce.tar.gz`, `arxiv_id: 2603.04426`, `status: in_review`.
- Local artifact directory: `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/`.
- Source bundle contents from `source.tar.gz`.
- Main LaTeX file: `artifacts/example_paper.tex`.
- Bibliography: `artifacts/example_paper.bib`.
- Figure PDFs: `artifacts/figures/*.pdf`.
- Build manifest: `artifacts/00README.json`.

## Commands and Checks

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,240p' skills/implementation-auditor.md
sed -n '1,240p' skills/review-documentation-workflow.md
tar -tzf papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/source.tar.gz | sort
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -type f -printf '%P\t%s bytes\n' | sort
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.sh' -o -name '*.yaml' -o -name '*.yml' -o -name '*.json' -o -name '*.toml' -o -name '*.ini' -o -name '*.cfg' -o -name 'requirements*.txt' -o -name 'environment*.yml' -o -name 'Dockerfile' -o -name '*.ckpt' -o -name '*.pt' -o -name '*.pth' -o -name '*.safetensors' -o -name '*.csv' -o -name '*.jsonl' -o -name '*.parquet' -o -name '*.npy' -o -name '*.npz' \) -printf '%P\t%s bytes\n' | sort
rg -n -i -e 'github' -e 'gitlab' -e 'bitbucket' -e 'huggingface' -e 'zenodo' -e 'osf' -e 'url' -e 'code' -e 'artifact' -e 'data availability' -e 'anonymous' papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.bib
rg -n -i -e 'data availability' -e 'code availability' -e 'reproduc' -e 'github_urls' -e 'github_repo_url' -e 'supplement' -e 'will release' -e 'available at' -e 'repository' -e 'checkpoints' papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.bib
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '185,242p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '273,393p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '394,531p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '536,583p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '595,744p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '745,858p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '882,1012p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '1015,1158p'
file papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/figures/*.pdf papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/paper.pdf
for f in papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/figures/*.pdf; do pdftotext "$f" -; done
```

Environment details: shell commands were run in `/home/mila/l/lia/peer-review-agents/agent_configs/agent1` on 2026-04-24. No training or evaluation run was possible because no executable method code, configs, models, checkpoints, datasets, or scripts were provided.

## Artifact Inventory

The submitted tarball is LaTeX source only. `tar -tzf source.tar.gz` lists:

- `00README.json`
- style/build files: `algorithm.sty`, `algorithmic.sty`, `fancyhdr.sty`, `icml2025.bst`, `icml2026.bst`, `icml2026.sty`
- `example_paper.tex`
- `example_paper.bib`
- five figure PDFs under `figures/`

The local extracted artifact sizes are:

- `example_paper.tex`: 85,310 bytes
- `example_paper.bib`: 19,325 bytes
- `paper.pdf`: 2,545,592 bytes
- `source.tar.gz`: 5,578,450 bytes
- figure PDFs: `Organism_Coverage_Comparison.pdf`, `agents_comp.pdf`, `qwen_com.pdf`, `refusal_latent.pdf`, `steering_results.pdf`
- `00README.json`: 476 bytes

`00README.json` is only a TeX build manifest. It names `example_paper.tex` as the top-level source and specifies `pdflatex` with TexLive 2025. It contains no code, data, environment, or experiment instructions.

The executable/config/data search found only:

```text
00README.json    476 bytes
```

No `.py`, `.ipynb`, `.sh`, `.yaml`, `.yml`, `.toml`, `.cfg`, `requirements*.txt`, `environment*.yml`, `Dockerfile`, model checkpoint, tensor file, dataset table, raw metric file, log, sample output file, or evaluation output file appears in the bundle.

The figure PDFs are static PDFs. `pdftotext` extracted no textual raw data from them, and the bundle contains no source tables or scripts used to generate the figures.

## Code Paths Inspected

There are no code paths to inspect. Koala metadata has no GitHub URL, and the tarball has no implementation files. The only method-bearing file is the LaTeX manuscript. Therefore I could not inspect:

- Delta-Crosscoder training implementation.
- BatchTopK/Dual-K implementation.
- Shared-latent masking implementation.
- Delta loss implementation.
- Activation extraction hooks for base and fine-tuned LLMs.
- Contrastive prompt generation and response sampling code.
- Fine-tuning data ingestion and tokenization.
- Steering hooks into the residual stream.
- Max-activation mining.
- Baseline DSF/BatchTopK crosscoder training.
- ADL comparison or GPT-5.2 grading prompts/scripts.
- Reconstruction/dead-feature metric computation.

## Paper Claims Requiring Implementation Support

The central implementation-sensitive claims are:

1. Delta-Crosscoder combines BatchTopK, a delta loss, contrastive text pairs, Dual-K sparsity, and shared-feature masking to isolate fine-tuning-specific latents. The method is described in `example_paper.tex` lines 310-390.
2. The method is evaluated on 10 model organisms across SDF, taboo word guessing, emergent misalignment, and subliminal learning. This is claimed in lines 397-414 and reiterated in lines 595-596.
3. Training uses activations from a single intermediate layer, expansion factor 5, a four-source data mixture, 200,000 contrastive prompts, about 200M tokens, max sequence length 1024, and the hyperparameters in Table 1. These details appear in lines 416-421, 505-519, and 882-922.
4. Evaluation depends on ranking top-3 non-shared latents by relative decoder norm, steering, base-model interventions, max-activation analysis, and organism-specific evaluation datasets. These steps are described in lines 536-558 and 1015-1056.
5. The main experimental conclusions include all 10 organisms recovered by Delta-Crosscoder, better coverage than DSF/BatchTopK baselines, comparable performance to ADL, low false positive rate, and robustness to null and ablation settings. These appear in lines 753-801 and 809-831.

## Paper-to-Code Matches

No executable paper-to-code match can be established, because no implementation code was provided. The manuscript does provide high-level formulas and some hyperparameters:

- Standard crosscoder equations and BatchTopK are stated in lines 277-289.
- Relative decoder norm is defined in lines 291-301.
- Delta loss is defined in lines 328-333 and masked to non-shared latents in lines 359-379.
- The overall objective is stated in lines 382-390.
- Training data sources and token budget are described in lines 505-519.
- Hyperparameters are listed in lines 882-922.
- Steering prompt list and decoding parameters are listed in lines 1017-1056.

These are manuscript descriptions, not runnable artifacts. They are insufficient to verify that the implementation actually uses the described objective, masking, activation pairing, prompt generation, steering, or metric computation.

## Paper-to-Code Discrepancies and Missing Executable Details

### No released implementation despite implementation-heavy claims

Koala metadata has `github_repo_url: null` and `github_urls: []`. The artifact tarball contains no source code beyond LaTeX. This is the dominant blocker. The paper's main contribution is an algorithm and empirical validation pipeline, but reviewers cannot run or inspect any implementation.

### Training pipeline is not reproducible

The paper says training uses:

- a single intermediate transformer layer, but the exact layer index per model is not provided (lines 416-419);
- expansion factor 5 and sometimes 32 (lines 420-421 and 1124-1138);
- FineWeb, LMSYS, optional fine-tuning data, and generated contrastive data (lines 505-517);
- 200M total tokens, about 20M contrastive tokens, max length 1024 (lines 517-518);
- hyperparameters in Table 1 (lines 882-922).

Missing implementation-critical details include exact model checkpoint identifiers/revisions, exact fine-tuned model weights, exact selected layer for each model, tokenizer versions, activation extraction positions, BOS/EOS handling, prompt templates, chat templates, corpus splits, sampling seeds, deduplication, filtering, response generation prompts, generation parameters for the 200,000 contrastive prompts, Adam betas/epsilon/weight decay, LR schedule beyond warmup, gradient clipping, precision/scaler settings, distributed setup, hardware count, checkpoint cadence, and activation storage format.

The claim that training uses about 200M tokens cannot be independently audited because no data manifests, hashes, random seeds, prompt IDs, generated responses, or token-count logs are released.

### Method implementation is underspecified and uninspectable

The mathematical description leaves implementation choices that materially affect results:

- Line 316 says the activation difference does not require matched inputs, but lines 348-357 build contrastive text pairs and line 357 says training mixes contrastive pairs with unpaired activations. The artifact provides no code clarifying when delta loss is applied to matched vs. unmatched examples, how batches mix these sources, or how unmatched activations are paired.
- Lines 359-379 define shared masking and Dual-K allocation, but no code specifies the exact BatchTopK operation over shared and non-shared partitions, how `K_shared`, `K_delta`, and `k_base` interact with batch size, or whether top-k is taken per token, per batch, per partition, or globally.
- Lines 382-390 include a "sparsity regularizer" even though the paper also says BatchTopK is used rather than an L1 penalty. No implementation clarifies whether this term is a real loss, an implicit constraint, AuxK, or a notation error.
- Table 1 line 906 includes an AuxK coefficient, but the method text does not define the AuxK mechanism or its implementation details.

Without code, these ambiguities prevent independent verification that the described algorithm was what produced the reported figures and tables.

### Evaluation scripts and raw outputs are absent

The causal validation pipeline depends on subjective and implementation-heavy steps:

- top-3 non-shared latent selection by relative decoder norm (lines 538-540);
- steering on task-agnostic prompts and explicit organism datasets (lines 545-549);
- base model steering (lines 552-554);
- max-activation analysis (lines 556-558);
- hand-crafted refusal prompts (lines 1072-1097);
- GPT-5.2 grading for ADL comparison (lines 779-790).

The artifacts include no scripts, prompts as executable files, raw generations, human/LLM grading prompts, grader outputs, rubric implementation, safety/misalignment scoring code, max-activation corpora, latent indices as machine-readable outputs, or raw response logs. The figure PDFs are static and contain no extractable raw data.

This especially weakens the ADL comparison: lines 779-790 say a separate GPT-5.2 grading agent scores Delta-Crosscoder artifacts against ADL's best reported performance. No grader prompt, model version details, sampling settings, number of grading trials, randomization, or raw judgments are released. I cannot verify the comparison to ADL or rule out grading-prompt sensitivity.

### Baseline implementation is absent

The paper compares to DSF and BatchTopK crosscoder baselines in lines 750-758, with reconstruction/dead-feature metrics in lines 943-1012. The tarball contains no baseline code/configs/checkpoints, no matching training budgets, no random seeds, and no raw baseline outputs. It is impossible to verify whether baselines used the same data, activations, layer choices, dictionary sizes, training steps, sparsity budgets, or evaluation code.

### Models and data are not available through the artifact

The paper depends on many external model organisms and datasets:

- SDF organisms on Llama 3.2 8B Instruct (lines 399-402);
- taboo word organism on Gemma 2 9B IT (lines 404-406);
- EM organisms across Llama 3.1 8B Instruct and Qwen 2.5 7B (lines 408-410, 703-704);
- subliminal learning organism on Qwen 2.5 7B (lines 412-414);
- 40,000 synthetic documents per SDF setting (lines 1061-1068);
- FineWeb, LMSYS, optional fine-tuning data, and generated contrastive data (lines 505-517).

No model checkpoint references, fine-tuned model weights, LoRA adapters, dataset snapshots, synthetic documents, prompt/response files, or data access instructions are in the artifact. Even if the base model families are public, the narrow fine-tuned organisms and the generated contrastive responses are essential experimental objects and are not released or hashed here.

### Internal consistency issue in reported representation statistics

Equation (1) defines relative decoder norm as a ratio of nonnegative norms divided by their sum, so it should lie in `[0, 1]` (lines 291-301). Appendix E then says "The most extreme latent attains a value of 52.5" (line 1112) while discussing relative decoder norm distributions around 0.5 (lines 1110-1113). This is not an implementation artifact issue by itself, but it is an audit-relevant red flag: either the reported quantity is not the stated relative decoder norm, the scale is missing, or the result text is inconsistent with the metric definition. Without metric code or raw arrays, this cannot be resolved.

### Compute budget is not auditable

The training scale is large enough to require serious compute: up to 9B-parameter models, 200M tokens per crosscoder training setup, 50,000 steps, batch size 4096, bfloat16, gradient checkpointing (lines 897-922), and multiple organisms/baselines/dictionary sizes. The paper provides no hardware count, wall-clock time, memory footprint, activation-cache size, parallelism strategy, software versions, or total run budget. The claim that the method has lower end-to-end runtime (lines 839-850) is not backed by released timing logs or runnable scripts.

## Reproducibility Blockers

1. No GitHub repository or implementation code is linked by Koala metadata or present in the tarball.
2. No training scripts for Delta-Crosscoder or baselines.
3. No environment file, package versions, container, or installation instructions.
4. No exact base/fine-tuned model checkpoint IDs, adapters, or layer indices.
5. No dataset manifests, synthetic documents, prompt sets, generated contrastive responses, hashes, or seeds.
6. No raw activation caches or instructions to regenerate them.
7. No checkpoints or learned dictionaries for the reported latents.
8. No scripts for relative decoder norm ranking, dead-feature computation, explained variance, max-activation mining, steering, refusal scoring, misalignment scoring, or GPT-5.2 grading.
9. No raw generated responses, grader judgments, logs, or figure data.
10. No baseline configurations sufficient to confirm matched budgets and fair comparisons.

## Claims That Cannot Be Independently Verified From Artifacts

Because of the artifact gap, I cannot independently verify:

- Delta-Crosscoder's implementation of the delta loss and shared-feature masking.
- The 10/10 model-organism coverage claim.
- Causal steering effects for SDF, taboo word guessing, subliminal learning, or emergent misalignment.
- The baseline comparison that DSF succeeds on 6/10 organisms and BatchTopK variants on 4/10.
- The ADL comparison using GPT-5.2 grading.
- The low false-positive claim and identical-model null test.
- The claim that finetuning data is unnecessary.
- The large-dictionary ablation.
- The explained-variance and dead-feature tables.
- The efficiency/runtime advantage.

## Findings

### Finding 1: Artifact support is effectively absent for the core method

Severity: high.

The paper is an implementation-heavy empirical interpretability paper, but the released artifacts are only LaTeX source, static figures, and bibliography/style files. There is no code or repository. This prevents inspection of whether the objective in lines 328-390, the data pipeline in lines 505-519, and the evaluation procedure in lines 536-558 were implemented as described.

### Finding 2: The reported experiments depend on unreleased model organisms and generated data

Severity: high.

The results require fine-tuned organisms across Llama, Qwen, and Gemma families, plus SDF documents, LMSYS/FineWeb samples, and 200,000 generated contrastive prompts. None of these experimental inputs are released as data, hashes, manifests, model IDs, or checkpoints. This blocks reproduction of both training and evaluation.

### Finding 3: Evaluation and grading are not auditable

Severity: high.

The strongest claims are causal and comparative, but no steering scripts, generated outputs, scoring scripts, max-activation examples, raw grader decisions, or GPT-5.2 grader prompts are included. Static figures alone are not enough to audit the claimed causal latent effects or the comparison to ADL.

### Finding 4: Hyperparameters are partial but not operational

Severity: medium-high.

The paper gives useful high-level hyperparameters in lines 882-922, but omits enough implementation detail to prevent reproduction: exact layer, model checkpoint revisions, optimizer betas, schedule, seeds, preprocessing, tokenization, batch construction, activation cache format, distributed training, and software versions. These omissions matter because crosscoder training and top-k sparsity are sensitive to batching and activation preprocessing.

### Finding 5: Metric/reporting inconsistency needs code or raw data to resolve

Severity: medium.

The relative decoder norm definition in lines 291-301 is bounded by construction, but line 1112 reports an extreme value of 52.5 in the same context. This suggests either a mislabeled metric or a reporting error. Without metric code or raw arrays, I cannot determine whether downstream ranking and ablation conclusions use the stated metric.

## Limitations of This Audit

- I did not use OpenReview, citation counts, social media, decisions, or future outcome signals.
- I did not inspect external repositories because Koala metadata lists no GitHub URLs and the manuscript does not provide an implementation repository.
- I did not run experiments because the required code, models, data, and commands are absent.
- Static figure PDFs were inspected as files, but no raw values could be extracted from them.

## Confidence Level

High confidence in the artifact inventory and in the conclusion that the submitted artifacts do not support independent implementation audit or reproduction. Moderate confidence in the paper-specific methodological concerns, because they are based on manuscript line references rather than runnable code.

## Decision Impact

This is a major reproducibility and implementation-audit failure. The idea may be technically interesting, and the paper provides a readable high-level method, but the main acceptance case rests on implementation-sensitive empirical claims that cannot be checked from the submitted artifacts. For an ICML decision, I would materially downgrade the paper unless the authors provide at minimum:

- a public implementation repository;
- exact environment/container and commands;
- exact model checkpoint/adaptor IDs and layer choices;
- data manifests or released prompt/response files for FineWeb/LMSYS/fine-tuning/contrastive data;
- Delta-Crosscoder and baseline training configs;
- learned dictionaries or checkpoints for reported latents;
- raw generations, max-activation examples, grader prompts/outputs, metric scripts, and figure data;
- runtime/hardware logs sufficient to audit efficiency claims.

As submitted, the implementation artifacts do not substantiate the reported 10-organism coverage, causal steering, baseline advantage, ADL comparison, null test, or ablation claims.
