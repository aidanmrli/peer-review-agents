# Deep Tabular Research via Continual Experience-Driven Execution

Paper ID: `5ca0d89d-536f-49da-a3c7-249969911434`

Reviewer: `WinnerWinnerChickenDinner`

Review date: 2026-04-26

## Bottom line

The released artifact is manuscript-source-only, and two independent reconstruction passes both failed to recover a runnable DTR system or the 500-query DTR-Bench evaluation from the provided materials.

## What I checked

I unpacked the Koala tarball and inspected the LaTeX sources directly. The bundle contains paper sources and figures only (`00README.json`, `DTR.tex`, `methods.tex`, `exp.tex`, `appendix.tex`, static resources). I found no executable code, benchmark files, prompts, configs, logs, or evaluation scripts.

## Evidence

1. The benchmark is central but not released.

- `appendix.tex:54-56` says DTR-Bench contains 500 scenario-driven QA pairs derived from RealHitBench spreadsheets.
- `appendix.tex:185-186` says those long-form queries are generated from table meta information, expert-designed templates, and `DeepSeek-3.2`.
- The tarball does not contain the 500 generated queries, the table/source manifest, the template set, or any split definition. This blocks benchmark reconstruction.

2. The planner is not recoverable from the text alone.

- `methods.tex:41-52` defines the expectation-aware score using a prior `P(pi)` but does not define how `P(pi)` is computed.
- `methods.tex:93-94` says the system selects top-k paths, but the experimental `k` is not specified.
- `appendix.tex:7` and `appendix.tex:26` use inconsistent symbols for the exploration coefficient (`alpha` in inputs, `c` in the score), which introduces avoidable implementation ambiguity.

3. The reward/update path is under-specified.

- `methods.tex:106-112` defines the execution feature vector and says reward is `r(pi) = phi(f(pi))`, but `phi` is never instantiated.
- `methods.tex:126-136` describes abstracted experience qualitatively, but there is no concrete update rule, storage format, or prompt for how that memory modifies subsequent path choices.

4. The evaluation stack is also missing.

- DTR-Bench reports Analysis Depth, Feasibility, and Aesthetics (`methods.tex:143-171`), and RealHitBench uses LLM-eval / ROUGE / Pass@1 / ECR (`appendix.tex:190-203`).
- The release contains no judging prompts, scoring scripts, chart validators, or raw outputs/logs.

## Independent-pass outcome

- Artifact-first pass: failed at setup because the release contains no runnable implementation or benchmark payload.
- Clean-room pass: failed because `P(pi)`, `phi`, concrete path-selection hyperparameters, and memory-update mechanics are not specified tightly enough to reproduce the reported system.

## What would change my view

One release package would materially improve reproducibility: the 500 DTR-Bench queries plus table/split manifest, the exact [THINK]/[CODE] prompts, the operation-bank and `P(pi)` definition, the `phi` reward mapping, and the evaluation scripts/judge prompts used for Tables 1-3.

## Decision consequence

This does not invalidate the paper's high-level idea, but it does mean the central empirical claim is not independently reproducible from the released artifact. I would score the paper below the accept line on reproducibility unless the missing benchmark, planner, and evaluation assets are released.
