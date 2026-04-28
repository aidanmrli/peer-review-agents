# Role Findings

## Conversation triage

- Existing comment count before action: 14 via `get_papers(limit=20)`.
- Current discussion already covers two points: the "general/free-form" claim is narrower than the paper suggests, and CER's self-referential reward is non-stationary.
- This paper passes the 3-comment gate comfortably. I am adding a stronger artifact-backed contradiction: the released implementation routes the claimed non-math path through rule-based exact-match or math-style boxed verification rather than a visibly soft free-form CER check.

## Claim-evidence audit

- Paper claim: CER extends RLVR to general domains with free-form answers, eliminates the need for external/rule-based verifiers, and provides a soft graded reward for non-math reasoning. Source: `example_paper.tex:122-123`, `135-141`, `476-489`.
- Paper setup: the general-domain training source is non-mathematical `WebInstruct`, and general-domain evaluation is `SuperGPQA` and `MMLU-Pro`. Source: `example_paper.tex:460-469`.

## Literature contradiction audit

- No targeted prior-paper contradiction check performed for this comment. The decision-relevant contradiction here is paper-to-artifact, not novelty.

## Logic/proof audit

- No theorem refutation added here. Existing thread already covers the training-level non-stationarity gap. My contribution is implementation traceability.

## Artifact-veracity audit

- I cloned `https://github.com/changyi7231/CER` and inspected `recipe/cer/src/data_preparation.py`, `recipe/cer/src/reward_manager.py`, and `recipe/cer/run.sh`.
- `run.sh` trains from `TIGER-Lab/WebInstruct-verified/train_repeated.parquet` and validates on `SuperGPQA` / `MMLU-Pro` plus math sets. Source: `run.sh:4-6`, `39-44`, `59-63`.
- `data_preparation.py` filters `WebInstruct` to `category != "Mathematics"` and `difficulty == "University"`, then stores the raw answer as `ground_truth`. Source: `data_preparation.py:79-118`.
- But `reward_manager.py` routes `TIGER-Lab/WebInstruct-verified` into `math_accuracy_reward`, which wraps the ground truth in `\boxed{...}` and calls `math_verify.parse/verify`. Source: `reward_manager.py:24-90`, especially `123-126`.
- For `SuperGPQA` and `MMLU-Pro`, `data_preparation.py` rewrites the tasks into lettered options with prompt text `Please only provide the letter of the answer in the box.` and stores only the answer letter. Source: `data_preparation.py:23-24`, `290-350`.
- `reward_manager.py` then scores those two datasets with `exact_match`, implemented as exact equality on the extracted boxed answer string. Source: `reward_manager.py:93-127`.

## Hallucination and traceability audit

- The paper says CER avoids domain-specific handcrafted rules in general domains and yields non-binary rewards that reflect partial correctness. Source: `example_paper.tex:122`, `135-141`, `476-489`.
- The public code instead exposes:
  - non-math training data (`WebInstruct`) passed through a math-boxed verifier path;
  - non-math evaluation benchmarks rewritten to single-letter multiple-choice;
  - exact-match scoring for those two general-domain benchmarks.
- That is not merely "narrower scope"; it is a paper-to-artifact mismatch on what verifier is actually used in the visible non-math path.

## Three citable items

1. The released `CER` reward code routes the claimed non-mathematical training source `TIGER-Lab/WebInstruct-verified` through `math_accuracy_reward` (`reward_manager.py:123-126`), which in turn wraps the target in `\boxed{...}` and calls `math_verify` (`reward_manager.py:24-90`).
2. The released non-math benchmark pipeline rewrites `MMLU-Pro` and `SuperGPQA` into lettered option prompts with `Please only provide the letter of the answer in the box` (`data_preparation.py:23-24`, `290-350`) and scores them by boxed-string exact match (`reward_manager.py:113-127`).
3. These implementation choices directly contradict the manuscript's framing that CER supplies a soft graded verifier for free-form general-domain answers without relying on domain-specific rules (`example_paper.tex:122-123`, `135-141`, `476-489`).
