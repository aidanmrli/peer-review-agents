# DIVE role findings

Paper: `c8877e38-1784-4b7f-a23a-a79a154ba733`
Title: `DIVE: Scaling Diversity in Agentic Task Synthesis for Generalizable Tool Use`
Reviewer: `BoatyMcBoatface`
Timestamp: `2026-04-26T13:47:00Z`

## Reproducibility lead

Central claim and reproduction target: verify whether the released artifact is sufficient to reproduce the paper's claimed training recipe and headline result path: 114k task pool, 48k SFT trajectories, 3.2k RL frontier tasks, Qwen3-8B fine-tuning, and the 9-benchmark evaluation.

## Reproducer A

Artifact-first check:
- The Koala tarball is paper-source only: `000abstract.tex`, `040recipe.tex`, `050experiment.tex`, appendix/tables/figures, and no runnable code.
- The author project page is live and links to public GitHub, dataset, and model pages.
- The GitHub repo is nontrivial: Python package (`dive/*.py`), 373-tool implementations under `tools/`, configs, seeds, exemplars, tests, docs, and a runnable CLI (`dive --config ... synthesize/end2end`).
- The Hugging Face dataset/model pages for `DIVE-SFT-20K` and `DIVE-8B-RL` are live.

Outcome: partial artifact recovery succeeded. This is materially better than manuscript-only release, but it is not yet enough to recreate the full paper recipe as written.

## Reproducer B

Clean-room/specification check:
- The manuscript states training uses a 114k task pool, 48k SFT trajectories via GPT-OSS-120B rejection sampling, and 3.2k RL tasks selected from a separate 38k pool (`050experiment.tex:16-17` in the unpacked tarball).
- The public README advertises `DIVE-SFT-20K`, `DIVE-RL-3K`, `DIVE-Eval`, and `DIVE-8B-RL`, not the full 48k/3.2k-from-38k recipe.
- The runnable config requires multiple live services and credentials: Anthropic/OpenAI-compatible LLMs, Serper, Jina, Tushare, plus optional Semantic Scholar/NCBI.
- The paper says all tool executions are performed against live tools, so rerunning synthesis is stateful and time-sensitive even with the code.

Outcome: a clean-room rerun of the exact paper pipeline is blocked by release mismatch and live-service dependence.

## Implementation auditor

Code/artifact/repo match:
- Repo README claims "Release code, data, and model for DIVE" and does expose code/data/model links.
- Public code includes prompt-bearing synthesis/verifier machinery and domain tool implementations, so the artifact is not vapourware.
- But the released data sizes do not match the manuscript's training recipe. I did not find a frozen 114k task pool, the 38k RL candidate pool, or a manifest tying the public subsets to the paper tables.
- I also did not find a credential-free replay path for the live-tool synthesis traces used to build the training data.

## Correctness specialist

Methods/metrics/conclusion risks:
- Main correctness issue here is reproducibility, not a math derivation bug.
- Because synthesis depends on live tools and provider models, the same code can yield drifted data unless the trace pool is frozen and released.
- This limits how strongly I can credit the scaling-law and benchmark claims as independently auditable.

## Literature specialist

Novelty/framing against permitted prior work:
- The current thread already covers OOD framing, exemplar leakage, APIGen/ToolACE, and teacher-distillation confounds well.
- My contribution is narrower: the artifact release partially resolves "no code" concerns, but the release still does not fully support reproduction of the exact recipe the paper reports.
