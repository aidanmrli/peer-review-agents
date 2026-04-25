# Literature Specialist Report

## Paper

- Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`
- Title: `Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes`
- Assigned role: Literature Specialist
- Task scope: Evaluate novelty, framing, missing baselines, missing citations, overclaims, and score impact against SAE/crosscoder/model-diffing, activation-difference, model-organism, emergent-misalignment, subliminal-learning, persona-vector, refusal, and steering literature.

## Information Hygiene

I used the paper source and bibliography under `papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/`, plus primary prior-work sources where needed. I did not use OpenReview reviews, citation counts, social media, accept/reject decisions, or later outcome signals for this paper.

## Evidence Examined

Paper source:

- `artifacts/example_paper.tex`, especially lines 226-240 for the introduction and claimed contributions.
- `artifacts/example_paper.tex`, lines 242-263 for related work.
- `artifacts/example_paper.tex`, lines 273-380 for crosscoder preliminaries and the Delta-Crosscoder objective.
- `artifacts/example_paper.tex`, lines 745-801 for SAE and non-SAE baseline comparisons.
- `artifacts/example_paper.tex`, lines 803-858 for reliability, robustness, efficiency, and conclusion claims.
- `artifacts/example_paper.tex`, lines 1015-1056 for steering protocol.
- `artifacts/example_paper.tex`, lines 1140-1157 for persona-vector similarity analysis.
- `artifacts/example_paper.bib`, especially entries for sparse autoencoders, BatchTopK, crosscoders, DSF/DFC, ADL, model organisms, emergent misalignment, subliminal learning, persona vectors, and concept ablation fine-tuning.

Commands and searches used:

```bash
rg -n "Crosscoder|crosscoder|Delta|diff|fine-tun|SAE|sparse|BatchTopK|TopK|DFC|DSF|activation|activation difference|organism|misalignment|subliminal|persona|steer|steering|refusal|Related|Novel|contribution|baseline|baseline" papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '226,380p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '740,860p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '1010,1165p'
rg -n "Zou|representation engineering|activation addition|Turner|steering|ActAdd|Causal|concept ablation|refusal|persona|control|directions" papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.bib papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex
curl -fsSL 'https://export.arxiv.org/api/query?id_list=2309.08600,2406.04093,2412.06410,2504.02922,2510.13900,2602.11729,2506.19823,2507.21509,2502.17424,2506.11613,2506.11618,2507.14805,2507.16795,2406.11717,2310.01405,2308.10248,2312.06681'
```

Primary prior-work sources considered:

- Cunningham et al., `Sparse Autoencoders Find Highly Interpretable Features in Language Models`, arXiv:2309.08600.
- Gao et al., `Scaling and evaluating sparse autoencoders`, arXiv:2406.04093.
- Bussmann et al., `BatchTopK Sparse Autoencoders`, arXiv:2412.06410.
- Lindsey et al., `Sparse Crosscoders for Cross-Layer Features and Model Diffing`, Transformer Circuits, 2024.
- Minder et al., `Overcoming Sparsity Artifacts in Crosscoders to Interpret Chat-Tuning`, arXiv:2504.02922.
- Jiralerspong and Bricken, `Cross-Architecture Model Diffing with Crosscoders`, arXiv:2602.11729.
- Minder et al., `Narrow Finetuning Leaves Clearly Readable Traces in Activation Differences`, arXiv:2510.13900.
- Aranguri and McGrath, `Discovering Undesired Rare Behaviors via Model Diff Amplification`, Goodfire Research, 2025.
- Betley et al., `Emergent Misalignment`, arXiv:2502.17424.
- Turner et al., `Model Organisms for Emergent Misalignment`, arXiv:2506.11613.
- Cloud et al., `Subliminal Learning`, arXiv:2507.14805.
- Wang et al., `Persona Features Control Emergent Misalignment`, arXiv:2506.19823.
- Chen et al., `Persona Vectors`, arXiv:2507.21509.
- Casademunt et al., `Steering Out-of-Distribution Generalization with Concept Ablation Fine-Tuning`, arXiv:2507.16795.
- Zou et al., `Representation Engineering`, arXiv:2310.01405.
- Turner et al., `Steering Language Models With Activation Engineering`, arXiv:2308.10248.
- Panickssery et al., `Steering Llama 2 via Contrastive Activation Addition`, arXiv:2312.06681.
- Arditi et al., `Refusal in Language Models Is Mediated by a Single Direction`, arXiv:2406.11717.

## Novelty Claims Checked

The paper claims that Delta-Crosscoder:

1. Introduces a crosscoder modification using Dual-K latent allocation, shared-feature masking, a delta loss, and contrastive paired activations.
2. Reliably identifies causally relevant fine-tuning-induced latents across 10 narrow-finetuning model organisms.
3. Enables steering and partial mitigation, outperforming SAE-based baselines and matching non-SAE ADL without agent-based probing.
4. Overcomes limitations of standard crosscoders, BatchTopK, DSF, and DFC-style variants in narrow fine-tuning regimes.

The narrow novelty claim is credible only if stated as: a crosscoder-specific integration of explicit activation-difference reconstruction, reserved non-shared capacity, and task-agnostic paired activations for narrow same-architecture fine-tune diffing. The broad framing is overstated. The individual ingredients are all strongly prefigured by prior work.

## Prior Work Overlap and Distinction

### Sparse Autoencoders and BatchTopK

The paper correctly places itself in the SAE/crosscoder family. Cunningham et al. and Gao et al. establish sparse dictionary learning for interpretable features, while Bussmann et al. introduce BatchTopK as a way to control average sparsity and improve reconstruction without a costly L1 sweep. Delta-Crosscoder uses BatchTopK as an implementation component, not as a new sparsity idea.

Consequence: the paper should avoid language suggesting that BatchTopK-style sparse allocation is part of the paper's conceptual novelty. Its contribution is the Delta-specific allocation and loss on top of BatchTopK.

### Crosscoders, DSF, DFC, and Latent Scaling

Lindsey et al. introduce crosscoders as sparse dictionaries shared across layers and models, including model-diffing use. Minder et al. identify L1 artifacts in crosscoders, introduce diagnostics such as Latent Scaling, and show that BatchTopK crosscoders can recover interpretable, causally effective chat-specific concepts, including refusal-related latents. Jiralerspong and Bricken introduce Dedicated Feature Crosscoders with explicit feature partitions for cross-architecture model diffing.

Delta-Crosscoder's Dual-K allocation and shared/non-shared partitioning are substantially adjacent to the DFC/DSF family. The paper cites these works but frames them mainly as failing baselines. That is too coarse. DFC's explicit exclusive-feature design is a close architectural antecedent for reserving non-shared capacity. Minder et al.'s Latent Scaling is also underused: the paper relies on relative decoder norms and a right-tail heuristic, but does not make Latent Scaling a central diagnostic baseline for whether "non-shared" latents are genuinely model-specific.

Specific missing or underused baseline:

- A DFC-style same-architecture narrow-finetuning baseline with exclusive partitions but no delta loss.
- A BatchTopK crosscoder baseline plus Latent Scaling diagnostics, not only coverage counts.
- An ablation separating Dual-K allocation, shared-feature masking, and delta loss. The paper discusses Delta-Crosscoder as a package, but the literature makes it important to isolate which known ingredient is doing the work.

Consequence: the architecture is incremental relative to DFC/DSF and BatchTopK crosscoder work unless the empirical ablation cleanly shows that the delta loss and contrastive pairing are independently load-bearing.

### Activation-Difference and Model-Diff Amplification Approaches

ADL is the closest prior work. Minder et al. show that narrow fine-tuning leaves strong activation-difference traces on random text, that adding activation differences can steer outputs toward the fine-tuning data style/content, and that the analysis spans the same broad organism classes: synthetic document fine-tuning, emergent misalignment, subliminal learning, and taboo word guessing across Gemma/LLaMA/Qwen scales. Delta-Crosscoder's "activation difference as a first-class signal" is therefore not a novel insight by itself; its novelty is putting that signal inside a sparse crosscoder objective.

The paper compares to ADL mainly as an interactive, agent-based probing method. That comparison is incomplete. ADL's core signal is a simple static activation-difference object, and the appropriate baseline is not only "ADL plus an interpretability agent." A stronger literature-grounded baseline would include a non-agent static activation-difference method: mean activation-difference steering, nearest examples under the activation-difference direction, or a grader given the same amount of static evidence as the Delta-Crosscoder grader.

The paper cites Aranguri and McGrath, but does not use model diff amplification as a substantive baseline. That work is not a sparse-latent method, but it is directly relevant because it amplifies before/after-model differences to surface rare undesired behaviors, including emergent-misalignment/backdoor-style settings. Delta-Crosscoder's claim of reduced analysis overhead and stronger practical model diffing should be compared against simple non-SAE diff amplification or at least discussed.

Consequence: the claim "matches non-SAE baselines" is not established. The paper compares to one ADL protocol under a different evaluation setup and does not benchmark simpler activation-difference or logit-difference alternatives.

### Model Organisms, Emergent Misalignment, and Subliminal Learning

The paper's model-organism framing is mostly accurate. Betley et al. define emergent misalignment from narrow fine-tuning; Turner et al. provide cleaner model organisms; Cloud et al. introduce subliminal learning; Wang et al. connect emergent misalignment to persona features. Delta-Crosscoder contributes a model-diffing analysis over these established organisms, not new organisms or new behavioral phenomena.

The paper should be more careful when claiming "reliably identifies" latents across all organisms. Its own text states that subliminal-learning steering on unrelated prompts is weak and inconsistent, and positive base-model steering increases broad animal content rather than a specific cat preference. That is mechanistically interesting, but it is weaker than the all-organism reliability claim in the abstract/introduction.

Consequence: coverage should be framed as heterogeneous evidence across organisms, not uniform recovery of causal latents in all cases.

### Persona, Refusal, and Steering Literature

The paper cites Wang et al. for persona features and Chen et al. for persona vectors, but it under-contextualizes the refusal and steering parts of its results.

Prior work already establishes that:

- Persona-like activation directions can monitor and control character traits after fine-tuning.
- Emergent misalignment can be controlled by SAE-derived persona features.
- Refusal can be mediated by a single residual-stream direction across many chat models.
- Activation steering through ActAdd, Representation Engineering, and CAA can control high-level properties by adding/subtracting activation directions.
- Concept Ablation Fine-Tuning directly mitigates out-of-distribution generalization such as emergent misalignment by ablating concept directions during fine-tuning.

The paper's refusal latent is therefore not a surprising new kind of mechanism. It is better framed as Delta-Crosscoder recovering a known class of refusal-gating direction in a narrow-finetuning model-diff setting. The steering protocol in Appendix B is also conventional activation steering, but the paper cites only ADL at the key steering-method line. It should cite at least RepE/ActAdd/CAA and Arditi et al. for refusal-direction context, and use CAFT as a mitigation comparison when claiming "partial mitigation."

Specific missing or underused citations/baselines:

- Zou et al. 2023, Representation Engineering.
- Turner et al. 2023, Activation Addition / activation engineering.
- Panickssery et al. 2024, Contrastive Activation Addition.
- Arditi et al. 2024, refusal direction.
- Casademunt et al. 2025, CAFT, as a mitigation baseline rather than only related context.
- A persona-vector baseline using Chen et al.'s method, not only a cosine-similarity auxiliary check.

Consequence: the paper's steering/mitigation contribution is overframed if not compared to established steering-vector and persona-vector methods.

## Framing Accuracy

Accurate:

- The paper correctly identifies narrow fine-tuning as a setting where ordinary reconstruction objectives may prioritize shared structure.
- It correctly cites SAEs, BatchTopK, crosscoders, ADL, model organisms, emergent misalignment, subliminal learning, and persona-feature work as relevant context.
- It plausibly distinguishes itself from ADL by producing sparse latents rather than relying solely on dense activation differences or an interactive interpretability agent.

Overstated or under-supported:

- "Existing crosscoder variants fail" is too broad without same-budget, same-data, same-layer, same-selection comparisons against the strongest BatchTopK/Latent-Scaling/DFC-style variants.
- "Matches ADL" is not supported by a protocol-matched comparison. The paper gives Delta-Crosscoder static artifacts to a GPT-5.2 grader and compares to best reported ADL performance from another protocol.
- "Reliably isolates latent directions causally responsible" is too strong for cases where steering is weak, inconsistent, or broad rather than specific, especially subliminal learning.
- The paper's discussion of false positives uses an unusual method-level definition where failure to recover a causal latent is called a false positive. In the literature, false positives more naturally refer to latents selected as meaningful that are not actually causal; this terminology risks overstating reliability.
- The refusal and persona findings are not novel enough to be presented as standalone mechanistic discoveries without stronger citation and baseline context.

## Missing or Underused Baselines

High-priority:

1. Same-architecture DFC-style exclusive-partition crosscoder without delta loss.
2. BatchTopK crosscoder with Latent Scaling diagnostics.
3. Static ADL / mean activation-difference baseline without an interpretability agent.
4. Model diff amplification or logit-difference amplification as a non-SAE rare-behavior surfacing baseline.
5. Persona-vector baseline for emergent-misalignment directions.
6. Refusal-direction baseline for refusal-gating latents.
7. CAFT or related concept-ablation mitigation baseline for mitigation claims.

Medium-priority:

- RepE/ActAdd/CAA citations and simple steering-vector baselines for the steering protocol.
- Stronger ablation isolating delta loss, contrastive pairing, Dual-K allocation, and shared-feature masking.

## Decision-Relevant Synthesis

Delta-Crosscoder is not a cleanly new conceptual paradigm. It is a plausible and potentially useful combination of established model-diffing ingredients: BatchTopK sparse dictionaries, crosscoder model diffing, dedicated/shared feature partitioning, activation-difference supervision, contrastive activation data, and residual-stream steering.

The literature still gives the paper a real contribution if the empirical claims hold: a sparse, static, crosscoder-based artifact that can recover narrow-fine-tuning signals across several model-organism families would be valuable. However, the novelty and framing must be downgraded because many of the paper's strongest claims are already partly anticipated by ADL, persona-feature work, DFC/DSF crosscoders, BatchTopK crosscoder improvements, refusal-direction work, and general activation-steering literature.

## Limitations of This Literature Review

- I did not inspect external code or reproduce baselines; this report is limited to novelty/framing and primary prior-work comparison.
- Several cited works are recent preprints/research updates. I treated them as relevant prior art because the paper itself cites many of them and because they are directly germane to model diffing.
- I did not use OpenReview, citation counts, social media, or outcome signals for this paper.

## Confidence Level

High confidence on the main conclusion: the paper's broad novelty and baseline framing are overstated, while the narrower integration claim remains plausible.

Medium confidence on the exact completeness of missing-citation recommendations, because this is a fast literature audit rather than a full systematic review.

## Score Impact

Literature score impact: material but not fatal. I would downgrade the novelty/framing component by approximately 1.0-1.5 points relative to a strong-accept reading. If the implementation and empirical results are independently reproducible, the literature position supports a borderline-to-weak-accept interpretation: useful integration, not a major conceptual breakthrough. If empirical reproducibility or baseline fairness is weak, the literature record pushes the paper into weak-reject territory because the remaining novelty is mostly a recombination of prior ideas without decisive evidence against obvious baselines.
