# Correctness Specialist Report: GameVerse

Paper: `daffb195-27bd-4b96-9236-b01d6fc210d2`, "GameVerse: Can Vision-Language Models Learn from Video-based Reflection?"

Role: Correctness Specialist. This pass checked internal consistency, metric validity, and claim strength.

## Paper Claim Being Tested

I checked whether the paper's conclusions are supported by its own methods and evidence:

- The milestone scoring pipeline is scalable and avoids manual annotation.
- Progress is quantified purely from pixels.
- Video-based reflection reliably improves VLM gameplay.
- Failure-plus-tutorial reflection is a training-free analogue of RL plus SFT.
- The taxonomy and dual action space support the claimed benchmark conclusions.

## Sources and Files Checked

- `artifacts/Chapter/Introduction.tex`
- `artifacts/Chapter/Benchmark.tex`
- `artifacts/Chapter/Experiment.tex`
- `artifacts/Chapter/Discussion.tex`
- `artifacts/Chapter/Appendix/A.tex`
- `artifacts/Chapter/Appendix/B.tex`
- `artifacts/Chapter/Appendix/C.tex`
- `repos/GameVerse/README.md`
- `repos/GameVerse/docs/video_reflection_guide.md`
- `repos/GameVerse/src/agent_servers/video_reflection.py`
- `repos/GameVerse/src/agent_servers/base_server.py`

## Commands

```bash
rg -n "manual|without human|purely from pixels|Gemini-3|hallucination|Reflection|RL|SFT|semantic|GUI|taxonomy|fire-and-forget" artifacts/Chapter repos/GameVerse
nl -ba artifacts/Chapter/Benchmark.tex | sed -n '50,88p'
nl -ba artifacts/Chapter/Experiment.tex | sed -n '1,32p;78,88p;172,182p'
nl -ba artifacts/Chapter/Appendix/C.tex | sed -n '96,210p'
nl -ba repos/GameVerse/docs/video_reflection_guide.md | sed -n '1,120p'
```

## Correctness Findings

1. The "without manual annotation" framing is contradicted by the paper.

`Introduction.tex` line 74 claims milestone scoring avoids manual annotation. `Benchmark.tex` line 81 says all milestones are manually verified. `Experiment.tex` line 13 says all performance metrics were manually verified. `Appendix/C.tex` lines 149-165 says 10 human experts evaluated milestone hallucination and representativeness. The correct claim is reduced annotation burden, not no manual annotation.

2. "Purely from pixels" is too strong for the milestone metric.

The agent may act from screenshots, but the evaluation pipeline in `Appendix/C.tex` line 134 uses an advanced VLM with game knowledge, long-context video understanding, and web-search capabilities, plus top-ranked public walkthroughs. This is not a model-independent visual metric. It avoids internal game APIs, which is valuable, but the paper should not imply that the metric is purely visual in a narrow sense.

3. Milestone matching error is decision-relevant.

`Appendix/C.tex` lines 170-177 reports hallucination rates up to 17 percent for Civilization VI, 12.5 percent for Baba Is You, 11 percent for Scene Investigators, and 10 percent for RDR2. `Appendix/C.tex` lines 195-201 reports VLM-human differences for discrete agent videos up to 14.2 percent for Genshin Impact, 12.1 percent for Ace Attorney, and 10.5 percent for Plants vs. Zombies. These errors are large enough to affect scores in tasks where many models have low absolute progress.

4. The reflection gains are nonuniform and not backed by released raw statistics.

`Experiment.tex` line 25 states reflection gains drop from 4.4 percent in non-real-time games to 1.7 percent in real-time games and scale with model capability. `Experiment.tex` line 82 says GUI-mode gains are about 3.75 percent versus 8.7 percent in semantic mode. These are plausible findings, but the paper does not release per-run traces or significance tests. For a benchmark paper, the difference between a robust improvement and noisy prompt sensitivity matters.

5. The RL plus SFT analogy is rhetorically stronger than the method.

`Experiment.tex` line 177 states that failure reflection acts like RL and tutorial reflection acts like SFT, and that their combination is a training-free proxy for combining SFT and RL. The implemented mechanism is prompt-conditioned reflection injection, as described in `docs/video_reflection_guide.md` and `src/agent_servers/base_server.py`. No parameters are updated, no reward optimization is performed, and no supervised fine-tuning occurs. The analogy is acceptable as intuition, but not as a validated algorithmic equivalence.

6. The taxonomy is useful but not validated as cognitive.

`Benchmark.tex` line 52 defines image structure, temporal dynamics, and causal linearity. `Appendix/A.tex` gives difficulty factors. The taxonomy is a reasonable benchmark design grid, but the paper does not validate that these axes are orthogonal, cognitively grounded, or predictive. Claims that it precisely probes capability boundaries should be tempered.

7. Dual action space is a diagnostic design, not a new correctness contribution.

`Benchmark.tex` line 54 defines semantic and GUI actions. The repository acknowledges GUI action settings are based on FlashAdventure and UI-TARS (`README.md` lines 213-215). The semantic-vs-GUI comparison is valuable, but the paper should frame it as a controlled diagnostic rather than a novel action-space design.

## Reproduction Outcome

Outcome: central conclusions not independently validated.

This role did not reproduce the numerical claims. The correctness pass instead found that several stated claims are stronger than the paper's own evidence permits. The strongest issue is the contradiction between "without manual annotation" and the reported manual verification/human-expert scoring.

## Exact Paper Locations

- `Introduction.tex` lines 71-78: novelty, scoring, and RL/SFT framing.
- `Benchmark.tex` lines 52-54: taxonomy and dual action space.
- `Benchmark.tex` lines 73-81: VLM milestone scoring and manual verification.
- `Experiment.tex` lines 9-13: reflection setup and manually verified metrics.
- `Experiment.tex` lines 25, 82, 177: reflection-gain conclusions.
- `Appendix/C.tex` lines 102-134: tutorial source selection and milestone scorer.
- `Appendix/C.tex` lines 149-201: human-expert validation and VLM-human differences.
- `README.md` lines 213-215: prior-work dependence for gaming loop and GUI action settings.

## Literature References Used

This correctness pass used literature only as context where the paper/repo itself invokes it: FlashAdventure, UI-TARS, Cradle, VideoGameBench, Orak, LMGame-Bench, and the post-training analogy paper cited in `Experiment.tex`.

## Final Synthesis and Score Impact

The paper contains a credible benchmark idea but overstates several conclusions. The metric is not manual-free, not fully model-independent, and not yet robust enough to be treated as a settled benchmark standard without released raw judge/human labels. The reflection results are interesting but should be framed as preliminary prompt-conditioned adaptation. Score impact: major concern for claims about scalable evaluation, moderate concern for reflection framing, moderate concern for taxonomy/action novelty.
