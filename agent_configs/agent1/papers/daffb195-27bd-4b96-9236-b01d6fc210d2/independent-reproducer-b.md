# Independent Reproducer B Report: GameVerse

Paper: `daffb195-27bd-4b96-9236-b01d6fc210d2`, "GameVerse: Can Vision-Language Models Learn from Video-based Reflection?"

Role: Independent Reproducer B. This pass avoided executable setup and instead attempted source-level and artifact-level reconstruction of the empirical claims.

## Paper Claim Being Tested

I tested whether the paper/source artifacts and repository contain enough static evidence to independently reconstruct:

- The milestone scoring pipeline and its validation.
- The reported reflection gains and semantic-vs-GUI gap.
- The raw evidence behind the benchmark tables.
- The exact scoring model and data used for milestone extraction/matching.

## Sources and Files Checked

Paper artifacts:

- `artifacts/Chapter/Benchmark.tex`
- `artifacts/Chapter/Experiment.tex`
- `artifacts/Chapter/Appendix/B.tex`
- `artifacts/Chapter/Appendix/C.tex`

Repository artifacts:

- `repos/GameVerse/README.md`
- `repos/GameVerse/docs/video_reflection_guide.md`
- `repos/GameVerse/scripts/generate_reflection.py`
- `repos/GameVerse/scripts/generate_milestone.py`
- `repos/GameVerse/src/agent_servers/video_reflection.py`
- `repos/GameVerse/src/agent_servers/base_server.py`
- `repos/GameVerse/src/agent_client/configs/*/config.yaml`

## Commands and Observations

Searches:

```bash
rg -n "Gemini-3|gemini-2|manual|without human|hallucination|semantic|reflection|tutorial|milestone|action space" artifacts/Chapter repos/GameVerse
find repos/GameVerse -maxdepth 2 -type d | sort
find repos/GameVerse -maxdepth 3 -type f | sed -n '1,180p'
```

Key observations:

- `Benchmark.tex` line 73 says milestone scoring uses `Gemini-3-pro` to quantify progress purely from pixels.
- `Benchmark.tex` line 81 says all milestones are manually verified.
- `Experiment.tex` line 13 says all performance metrics were manually verified.
- `Appendix/C.tex` line 134 says the milestone VLM uses game knowledge, long-context video comprehension, and web-search capabilities, and describes procedural scores "without human intervention."
- `Appendix/C.tex` lines 149-177 reports hallucination rates and representativeness scores evaluated by 10 human experts.
- `Appendix/C.tex` lines 183-201 reports VLM-human matching comparisons, including 14.2 percent difference for Genshin Impact, 12.1 percent for Ace Attorney, and 10.5 percent for Plants vs. Zombies in discrete agent-frame matching.
- Public configs and environment defaults contain many `milestone_model: "gemini-2.0-flash-exp"` values, while the paper says `Gemini-3-pro`.
- `docs/video_reflection_guide.md` expects generated logs, `data/reflections/`, and `data/expert_videos/...`; these raw artifacts are not included in the clone.

## Reproduction Outcome

Outcome: not reproduced.

This pass could verify that the paper and repo describe a milestone/reflection workflow, but could not recover the paper's numerical results. The raw objects needed for source-level reconstruction are absent:

- No raw per-episode logs.
- No obs frame sequences or gameplay videos behind table values.
- No extracted milestone JSON files.
- No VLM judge prompts/responses as executed for the paper.
- No human expert verification labels.
- No table-generation script connecting logs to `Experiment.tex`.
- No seed/run list tying the reported standard deviations to individual trials.

The TeX files report final numbers, but static inspection cannot determine whether those numbers follow from the released code.

## Exact Paper and Repo Locations

- `Benchmark.tex` lines 73-81: milestone scorer and manual verification.
- `Experiment.tex` lines 7-13: evaluated models, baselines, and manually verified metrics.
- `Experiment.tex` lines 25, 82, and 177: reflection-gain, semantic-vs-GUI, and failure/tutorial conclusions.
- `Appendix/C.tex` lines 102-134: tutorial/walkthrough source selection and VLM milestone scorer.
- `Appendix/C.tex` lines 149-201: human-expert milestone evaluation and VLM-human matching scores.
- `docs/video_reflection_guide.md` lines 9-12 and 67-75: expected generated data and expert-video paths.
- `src/agent_servers/video_reflection.py` implements failure/expert video reflection, but not release-time evidence for the paper tables.

## Role Findings

The source artifacts support the existence of a benchmark framework, not the reproducibility of the paper's empirical claims. The paper itself acknowledges manual verification and nontrivial VLM judge error. That should have been paired with released raw judge and human-verification artifacts.

## Literature References Used

This pass did not use external literature. It focused on internal consistency between paper, source, and repository.

## Final Synthesis and Score Impact

Independent static reconstruction failed. This reinforces Reproducer A's execution-level conclusion: the main results are not reproducible from the released evidence. The paper should be scored down unless the authors provide the run traces, scoring artifacts, and aggregation scripts.
