# Literature Specialist Report

Paper: `c1935a69-e332-4899-b817-9c7462a4da4d`  
Title: "Consensus is Not Verification: Why Crowd Wisdom Strategies Fail for LLM Truthfulness"  
Role: Literature Specialist  
Date: 2026-04-24

## Central Novelty / Literature Claim Tested

I tested the paper's literature claim that prior work establishes (i) inference-time aggregation gains mainly where answers can be externally verified and (ii) correlated LLM errors are known, but leaves unresolved whether this correlation structurally prevents polling-style self-aggregation from improving truthfulness in verifier-absent domains. The relevant paper claims occur in the abstract and introduction (`main.tex` lines 92-94, 102-114), the contribution list (lines 118-122), the related work section (lines 131-146), the setup (lines 155-212), and the conclusion (lines 702-710).

## Prior Work Considered

Sources checked were the submitted LaTeX source and bibliography plus primary pre-release literature pages for directly relevant works. I did not use OpenReview reviews, decisions, citation counts, social media, later impact signals, or any source about this exact submitted paper.

Key cited work that supports the paper's framing:

- Classical crowd aggregation: Condorcet (1785), Surowiecki (2004), van Dolder and van den Assem (2018), Prelec et al. (2017). These are cited in `main.tex` lines 104, 133, 137, 267, 299 and bibliography lines 163-194. They support the paper's premise that crowd methods require partially independent errors or an expert-minority structure.
- Ensemble and test-time scaling: Lakshminarayanan et al. (2017), Wang et al. (2023), Brown et al. (2024), Schaeffer et al. (2025), Snell et al. (2025). These are cited in `main.tex` lines 102, 131, 135 and bibliography lines 112-125, 197-223. They support the contrast with settings where more samples can help.
- LLM error correlation and oversight concerns: Kim et al. (2025) and Goel et al. (2025), cited at `main.tex` line 135 and bibliography lines 262-274. These make the paper's correlated-error motivation well grounded rather than speculative.
- Confidence and sycophancy: Kadavath et al. (2022), Tian et al. (2023), Xiong et al. (2024), Sharma et al. (2023), Leng et al. (2025), cited at `main.tex` lines 139 and 278 and bibliography lines 135-140, 276-311. These adequately support the claim that verbalized confidence is not a reliable generic truth signal.
- Benchmarks: BoolQ, Com2Sense, HLE, and the paper's Predict-the-Future dataset are described in `main.tex` lines 161-166 and 812-838, with BoolQ/Com2Sense/HLE references in bibliography lines 10-16, 149-160.

Directly relevant prior work that is present in the bibliography but under-integrated or absent from the paper narrative:

- Schoenegger et al. (2024), "Wisdom of the Silicon Crowd," bibliography lines 28-37, reports that aggregating LLM forecasts can rival human crowd accuracy. This is a direct positive-result predecessor for LLM crowd forecasting, but `rg` finds no citation in `main.tex`.
- ForecastBench (Karger et al., 2025), bibliography lines 290-295, is a dynamic benchmark designed to avoid forecasting leakage and evaluate AI forecasting. It is not cited in `main.tex`, despite the paper introducing a small forecasting dataset and presenting it as the clearest negative test (`main.tex` lines 166, 211-212, 265, 681-683).
- Universal Self-Consistency (Chen et al., 2023), bibliography lines 98-103, explicitly studies self-consistency for open-ended generation without ordinary answer extraction and reports improvements on open-ended QA and other tasks. It is not cited in `main.tex`.
- Multiagent Debate (Du et al., 2024), bibliography lines 226-230, claims improvements in factual validity and reasoning through multiple LLM instances. It is not cited in `main.tex`, although it is directly relevant to the broad title-level claim that crowd wisdom strategies fail for LLM truthfulness.
- Multiple-choice selection bias work, especially Zheng et al. (2023), "Large Language Models Are Not Robust Multiple Choice Selectors," is missing from the bibliography. This prior work shows LLMs can prefer option IDs independent of content, which is directly relevant to the random-string forced-choice control in `main.tex` lines 110, 122, 321-339, and 400-429.

Commands / checks used:

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,220p' skills/literature-specialist.md
find papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts -maxdepth 3 -type f | sort
rg -n "(Related|Contribution|novel|crowd|consensus|verification|truth|self-consistency|debate|confidence|agreement|forecast)" papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex
rg -n "(schoenegger|forecast|universal|debate|correlated|goel|kim|tian|leng|xiong|sharma)" papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/references.bib
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '88,152p'
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '155,216p'
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '261,320p'
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/main.tex | sed -n '616,710p'
nl -ba papers/c1935a69-e332-4899-b817-9c7462a4da4d/artifacts/source/references.bib | sed -n '1,318p'
```

## Gaps, Overclaims, and Distinctions

The core negative result is not merely a rediscovery. The combination of verifier-absent binary truthfulness tasks, confidence/predicted-popularity/SP aggregation, cross-family error correlation, and a no-truth random-string control is a coherent and likely useful empirical package. The related-work claim at `main.tex` lines 145-146 is mostly defensible if narrowly read as: "given already-known correlated LLM errors, this paper tests whether common endogenous aggregation signals overcome them in verifier-absent binary tasks."

However, the broad framing is too strong in three places.

First, the statement that prior inference-time scaling successes share automatic verifiability (`main.tex` line 131; discussion lines 619-622 and 702-710) overcompresses the literature. Wang et al. self-consistency is itself a majority-style aggregation method and reported gains on arithmetic and commonsense reasoning tasks, not only code/proof-checker settings. Chen et al. universal self-consistency and Du et al. multiagent debate are even closer counterpoints because they explore LLM-based aggregation/debate without simple external execution. The paper can still argue that those settings differ, but it should explicitly separate "external verifier," "benchmark label available only for evaluation," "LLM-as-judge/implicit verifier," and "pure polling."

Second, the forecasting framing is incomplete. Predict-the-Future is presented as the clearest negative test (`main.tex` lines 211-212 and 265), but the paper does not discuss Schoenegger et al.'s positive "silicon crowd" forecasting results or ForecastBench's dynamic anti-contamination forecasting setup, even though both are in or near the submitted bibliography. This omission weakens the novelty and external-validity framing of the new forecasting benchmark. The paper needs to explain why its binary post-cutoff forecasting setting, model set, answer format, and aggregation rules differ from prior LLM forecast aggregation work that found positive crowd effects.

Third, the random-string control is a good diagnostic but is framed too quickly as evidence of deep shared inductive biases rather than shared knowledge (`main.tex` lines 110, 122, 429). Prior MCQ-selection-bias work already shows that option-token priors can drive answers independently of content. Without engaging that literature, the random-string result may partly measure known answer-label priors or chat-template decoding biases rather than a broader structural property of truthfulness.

There are also smaller framing issues:

- The abstract says "across five benchmarks and models" (`main.tex` line 92), while the main verifier-absent evaluation lists four benchmarks (`main.tex` lines 161-166, 263). MATH/AIME appear later as verifiable-domain analyses (`main.tex` lines 303-309), not as part of the main verifier-absent claim. This should be clarified.
- The paper claims the five listed aggregation rules "exhaust common internal selection signals" (`main.tex` lines 195-202). That is too broad given prior LLM-as-judge, debate, self-reward, universal self-consistency, and learned aggregator methods. It is accurate only for simple polling over answer, confidence, and predicted popularity signals in binary tasks.

## Reproducibility / Literature Impact

From a literature standpoint, the paper is strongest as a negative empirical diagnostic: it ties known correlated-error concerns to the failure of simple verifier-free aggregation rules and makes the social-prediction versus truth-verification distinction explicit (`main.tex` lines 653-663 and 708-710). That is decision-relevant and not fully subsumed by the cited literature.

The novelty claim should be downgraded from "crowd wisdom strategies fail for LLM truthfulness" to "simple polling-style self-aggregation over binary verifier-absent tasks fails under the tested models, datasets, and signals." Prior work already contains positive and mixed evidence for LLM ensembles, self-consistency, debate, and forecasting aggregation. The paper's contribution is therefore a scoped negative boundary result, not a general refutation of LLM crowd methods.

The implementation/reproduction reports for this paper note that no code, raw model outputs, benchmark item IDs, or Predict-the-Future data are included. That limitation matters for literature grounding too: because the paper's novelty rests on an empirical boundary claim against existing positive aggregation literature, the absence of released data/code prevents reviewers from checking whether the negative result is due to the conceptual regime, the binary reduction, the model family mix, prompt/parser choices, or the small forecasting benchmark.

## Final Score Impact

Literature impact: moderate negative. The paper asks an important question and has a plausible, useful niche, but its novelty and framing are overstated relative to directly relevant prior work on LLM forecast aggregation, universal self-consistency, multiagent debate, and multiple-choice selection bias. I would reduce the literature/novelty component materially, roughly a 0.5-1.0 point downgrade on a 10-point review scale. This literature issue is not a standalone clear-reject reason, but combined with the artifact-level reproducibility limits it prevents a strong-accept-level assessment.
