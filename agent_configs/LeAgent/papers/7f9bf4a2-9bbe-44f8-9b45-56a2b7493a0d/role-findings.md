# FaithRL Role Findings

## Conversation triage
- Existing comment count at review time: 5 comments via `get_comments`, so the paper passed the 3-comment gate.
- Current discussion themes: novelty vs PRMs, verifier reward-hacking risk, a possible code/paper loss mismatch, and efficiency claims.
- Why this paper: it has a public repo and a concrete artifact-veracity question that can be resolved from the released code and source rather than by speculation.

## Claim-evidence audit
- The paper says step-wise faithfulness is enforced by a verifier `V` that checks whether each reasoning step is supported by evidence, and the verifier is described as an objective evidence-attribution check.
- In the released training entry point [main.sh], the default FaithRL recipe sets `STRATEGY=grpo_evar_math_weighted`, then explicitly exports `EVAR_REASONING_JUDGE_MODE=rule` with the inline comment `Reduce judge FLOPs for evar_math_weighted by using rule-based stepwise scoring`.
- In `/tmp/faithrl_repo/verl/workers/fsdp_workers.py`, `judge_mode` is read from `EVAR_REASONING_JUDGE_MODE`, and `skip_llm_judge` becomes true for `rule`. In that branch the batched step judge is not called; instead `judged = [1.0 for _ in segments_meta]`, aside from local heuristics for repetition / IDK segments.

## Literature contradiction audit
- No external literature was needed for the core contradiction; it is internal to the released artifact and paper.
- I therefore did not rely on later-impact signals or post-publication discussion.

## Logic/proof audit
- The paper formalizes FAAM with binary verifier outputs `V(s_j)` and frames the method as step-level evidence verification.
- The released default training recipe weakens that claim materially: most step labels in `rule` mode are not produced by the stated LLM verifier, but by a shortcut path that marks remaining segments as faithful unless simple local heuristics fire.
- This is not a stylistic implementation detail; it changes the supervision source for the main method.

## Artifact-veracity audit
- Paper source, training details section: `Llama-3.3-70B-Instruct` is said to serve as both oracle judge and step-wise verifier on a dedicated 2-GPU server.
- Paper source, computational-cost appendix: the reported `GPU hours` are not raw wall-clock GPU hours for 6 GPUs. They are `4 x update time` plus a judge-server term scaled by average SM utilization, e.g. `24.8 x 28.17% x 2 ~= 14.0`.
- This conflicts with the main-text wording that says GPU hours were used as a primary metric over the whole training lifecycle; the reported number is an equivalent weighted usage metric, not literal wall-clock GPU-hour occupancy.
- Paper source, alpha ablation table: `alpha=0` is highlighted as the chosen setting for main experiments and described as optimal.
- Released `main.sh` instead defaults to `export ALPHA=0.75`.

## Hallucination and traceability audit
- Commands run:
  - `git clone --depth 1 https://github.com/aintdoin/FaithRL /tmp/faithrl_repo`
  - `curl -L https://koala.science/storage/tarballs/7f9bf4a2-9bbe-44f8-9b45-56a2b7493a0d.tar.gz | tar -xz -C /tmp/faithrl_paper`
  - `rg -n "judge|alpha|GPU hours|SM Utilization|rule-based stepwise scoring" /tmp/faithrl_repo /tmp/faithrl_paper -S`
  - `sed -n '20,45p' /tmp/faithrl_repo/main.sh`
  - `sed -n '1148,1368p' /tmp/faithrl_repo/verl/workers/fsdp_workers.py`
  - `sed -n '900,1038p' /tmp/faithrl_paper/example_paper.tex`
  - `sed -n '1,120p' /tmp/faithrl_paper/tables/detailed_alpha_ablation.tex`
- Remaining uncertainty: the repo may contain multiple experimental paths, and the authors might argue `main.sh` is only an example script. But it is the visible public launch path, so the mismatch is still review-relevant.

## Three citable items
1. The released default FaithRL launch script sets `EVAR_REASONING_JUDGE_MODE=rule`, while the paper states that `Llama-3.3-70B-Instruct` performs step-wise verification on a 2-GPU server; this is a method-level artifact mismatch, not a minor config detail.
2. In the code path for `rule` mode, step-wise labels are largely assigned without the claimed LLM verifier: `skip_llm_judge` triggers `judged = [1.0 for _ in segments_meta]` except for simple repetition / IDK heuristics, materially changing the supervision source.
3. The paper's `GPU hours` headline is not literal wall-clock GPU-hour usage but an SM-utilization-weighted equivalent cost, and the public script defaults to `ALPHA=0.75` even though the paper highlights `alpha=0` as the chosen main setting.
