# Reproducibility Lead Report: GameVerse

Paper: `daffb195-27bd-4b96-9236-b01d6fc210d2`, "GameVerse: Can Vision-Language Models Learn from Video-based Reflection?"

Role: Reproducibility Lead for `agent1`.

## Paper Claim Being Tested

The central reproducibility question is whether a reviewer can independently recover the paper's benchmark conclusions from the paper, source artifacts, and author-linked repository:

- GameVerse reports normalized VLM scores across 15 games, with and without video-based reflection.
- The paper claims VLMs improve under a failure-plus-tutorial "reflect-and-retry" loop.
- The paper claims scalable milestone evaluation from pixels, using advanced VLMs and minimal/no manual annotation.
- The paper claims the released benchmark supports semantic and GUI action modes across the game suite.

## Sources and Files Checked

Paper artifacts:

- `artifacts/GeneralGameBench.tex`
- `artifacts/Chapter/Introduction.tex`
- `artifacts/Chapter/Benchmark.tex`
- `artifacts/Chapter/Experiment.tex`
- `artifacts/Chapter/Discussion.tex`
- `artifacts/Chapter/Appendix/A.tex`
- `artifacts/Chapter/Appendix/B.tex`
- `artifacts/Chapter/Appendix/C.tex`
- `artifacts/Reference.bib`

Repository artifact:

- `repos/GameVerse` at commit `7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd`
- `README.md`
- `requirements.txt`
- `pyproject.toml`
- `scripts/play_game.py`
- `scripts/generate_reflection.py`
- `scripts/generate_milestone.py`
- `scripts/leaderboard/*`
- `docs/video_reflection_guide.md`
- `src/agent_client/configs/*/config.yaml`
- `src/game_servers/*/game/*_env.py`
- `src/agent_servers/video_reflection.py`
- `src/agent_servers/base_server.py`

Koala platform state:

- `get_paper(daffb195-27bd-4b96-9236-b01d6fc210d2)` returned status `in_review`, no platform `github_repo_url`, and empty `github_urls`, although the abstract links `https://github.com/THUSI-Lab/GameVerse`.
- `get_comments(...)` returned only bibliography-formatting comments, with no existing substantive reproducibility review.

## Commands and Environment

Working directory:

```bash
/home/mila/l/lia/peer-review-agents/agent_configs/agent1
```

Repository inspection:

```bash
git -C papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse rev-parse HEAD
# 7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd

find papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse -maxdepth 2 -type d | sort
find papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse -maxdepth 3 -type f | sed -n '1,180p'
rg -n "Gemini-3|gemini-2|manual|without human|hallucination|semantic|reflection|tutorial|milestone|action space" ...
```

Executable smoke tests:

```bash
cd papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse
python scripts/play_game.py --help
python scripts/generate_reflection.py --help
python scripts/generate_milestone.py --help
```

All three smoke tests failed in the current review environment before argument parsing with:

```text
ModuleNotFoundError: No module named 'omegaconf'
```

I did not install dependencies or game binaries during this pass. The failure is therefore a local environment blocker, not by itself a proof that the released repository cannot run. The stronger reproducibility issue is that the repository does not include raw benchmark runs, scored videos, extracted milestone JSON files, judge outputs, seeds, or table-generation artifacts needed to independently recompute the reported results.

## Role-by-Role Findings

Independent Reproducer A attempted to follow the repository's executable entry points. The documented scripts exist, but the local environment lacked dependencies, and the reviewer could not proceed to even help output without installing the stack. More importantly, the paper's reported tables cannot be reproduced from included raw data because the repository does not provide the underlying run logs, videos, model outputs, or exact API judge outputs.

Independent Reproducer B used a source-and-artifact route instead of an execution route. This pass found that the TeX tables contain reported means/standard deviations and milestone-validity statistics, but the artifact set lacks machine-readable raw observations behind them. The paper says metrics are manually verified, and Appendix C reports human-expert evaluation of milestone hallucination and matching differences; those human annotations are not released as auditable data.

The Implementation Auditor found a paper-code mismatch around milestone models. The paper states milestone scoring uses `Gemini-3-pro` (`Benchmark.tex` line 73; `Appendix/C.tex` line 134), while many public configs and environment defaults use `gemini-2.0-flash-exp` and some use `gemini-2.5-pro`. The repository also expects local `logs/`, `data/reflections/`, and `data/expert_videos/` workflows, but these evidence directories are absent from the release clone.

The Correctness Specialist found overstatements around "without manual annotation" and "purely from pixels." `Benchmark.tex` line 81 says all milestones are manually verified. `Experiment.tex` line 13 says all performance metrics were manually verified. `Appendix/C.tex` lines 149-201 reports 10 human expert evaluation, hallucination rates up to 17 percent, and VLM-human score differences up to 14.2 percent for discrete agent videos. `Appendix/C.tex` line 134 says the judge has extensive game knowledge and web-search capability.

The Literature Specialist found that the benchmark-integration contribution is real, but the framing overclaims novelty for reflection/retry, the taxonomy, the dual action space, and scalable milestone scoring relative to VideoGameBench, LMGame-Bench, Orak, Cradle, FlashAdventure, Voyager, ROE, GUI grounding benchmarks, and video/tutorial-learning work.

## Reproduction Outcome

Outcome: weak reproducibility for the central empirical claims.

Two independent internal passes failed to reproduce the central results:

- Reproducer A could not execute the documented scripts in the current environment and found no raw release artifacts to recompute the benchmark tables.
- Reproducer B could not reconstruct the paper's results from the source bundle because the tables are not backed by included raw logs, videos, extracted milestones, judge decisions, or table-generation scripts.

The repository establishes a plausible implementation skeleton and provides configs and leaderboard scripts for several games, but this is not enough to independently recover the paper's quantitative claims.

## Exact Paper Locations

- `Introduction.tex` lines 71 and 74: claims a novel reflect-and-retry paradigm and scalable milestone scoring without internal APIs or manual annotation.
- `Benchmark.tex` lines 52-54: taxonomy and dual action space.
- `Benchmark.tex` lines 73-81: Gemini-3-Pro milestone scoring and manual milestone verification.
- `Experiment.tex` lines 7-13: model list, reflection setup, baselines, and manually verified metrics.
- `Experiment.tex` lines 25 and 82: reflection gain and semantic-vs-GUI gap claims.
- `Experiment.tex` line 177: failure-plus-tutorial reflection framed as a training-free analogue of RL plus SFT.
- `Appendix/C.tex` lines 102-134: public tutorial/walkthrough selection and milestone scoring pipeline.
- `Appendix/C.tex` lines 149-201: milestone hallucination, human-expert evaluation, and VLM-human matching differences.
- `README.md` lines 118-168: evaluation and reflection/milestone scripts.
- `README.md` lines 213-215: gaming loop and GUI action space acknowledgements.
- `docs/video_reflection_guide.md` lines 9-12 and 67-75: expected logs/reflections/expert-video workflow.

## Literature References Used

Permitted prior work checked by the Literature Specialist:

- VideoGameBench
- LMGame-Bench
- Orak
- Cradle
- FlashAdventure
- Voyager
- Reflection of Episodes
- Reflexion
- Self-Refine
- R3V
- MineDojo
- GROOT
- OSWorld
- ScreenSpot-Pro
- UI-TARS
- V-MAGE
- AI GameStore

No OpenReview reviews, decisions, acceptance status, citation trajectories, or external commentary about this exact paper were used.

## Final Synthesis and Score Impact

GameVerse is a useful benchmark proposal with a substantial engineering surface, but the current release does not make its main quantitative claims independently reproducible. The lack of raw logs/videos/milestone JSONs/judge outputs/table scripts, plus paper-code mismatch on the milestone judge model and reliance on manual verification, should materially lower confidence. The appropriate public comment should request reproducibility artifacts and narrow the novelty claims to benchmark integration rather than novel reflection, novel action spaces, or fully automated evaluation.

Score impact: substantial negative on reproducibility and moderate negative on literature framing. The benchmark utility may still be positive if the authors release full traces and scoring artifacts, but current evidence supports at most a cautious weak-accept/weak-reject boundary rather than a strong accept.
