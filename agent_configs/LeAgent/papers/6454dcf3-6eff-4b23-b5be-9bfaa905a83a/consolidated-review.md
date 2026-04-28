# CER Artifact Check

Paper: `6454dcf3-6eff-4b23-b5be-9bfaa905a83a`  
Title: `Reinforcement Learning with Conditional Expectation Reward`

## Bottom line

The public `CER` implementation exposes a stronger contradiction than "the non-math scope is narrow." In the released code, the visible non-math path is not a soft free-form verifier at all: `WebInstruct` is routed through a math-style boxed-answer verifier, while `MMLU-Pro` and `SuperGPQA` are rewritten into single-letter multiple-choice tasks and scored by boxed exact match.

## Evidence

1. The manuscript says CER extends RLVR to general/free-form domains, eliminates reliance on external or rule-based verifiers, and supplies soft graded rewards. Relevant paper text:
   - `example_paper.tex:122-123`
   - `example_paper.tex:135-141`
   - `example_paper.tex:476-489`

2. The released training script uses non-math `WebInstruct` as the training source and validates on `SuperGPQA` / `MMLU-Pro`:
   - `recipe/cer/run.sh:4-6`
   - `recipe/cer/run.sh:39-44`
   - `recipe/cer/run.sh:59-63`

3. In `data_preparation.py`, `WebInstruct` is explicitly filtered to non-math university questions, then stored with raw answers as ground truth:
   - `recipe/cer/src/data_preparation.py:79-118`

4. But in `reward_manager.py`, that same `TIGER-Lab/WebInstruct-verified` source is scored by `math_accuracy_reward`, which wraps the target in `\boxed{...}` and calls `math_verify.parse/verify`:
   - `recipe/cer/src/reward_manager.py:24-90`
   - `recipe/cer/src/reward_manager.py:123-126`

5. For the paper's two "general-domain" evaluation benchmarks, the public pipeline rewrites examples into lettered options with the prompt:
   - `Please only provide the letter of the answer in the box.`
   Source:
   - `recipe/cer/src/data_preparation.py:23-24`
   - `recipe/cer/src/data_preparation.py:290-350`

6. Those two datasets are then scored by strict boxed-answer exact match:
   - `recipe/cer/src/reward_manager.py:113-127`

## Decision consequence

This does not prove the paper's numbers are false. It does mean the visible artifact does not match the paper's strongest non-math framing. The released implementation supports a much narrower claim:

- non-math training is passed through a math-style verifier path;
- non-math benchmark evaluation is multiple-choice exact match;
- the artifact does not expose a clearly auditable soft free-form CER verifier for general-domain answers with multiple valid surface forms.

That lowers confidence in the manuscript's repeated claim that CER itself is what establishes broad free-form generality on the released non-math path.
