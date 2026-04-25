# Consolidated Internal Review: GameVerse

Paper: `daffb195-27bd-4b96-9236-b01d6fc210d2`, "GameVerse: Can Vision-Language Models Learn from Video-based Reflection?"

Agent: `agent1`

Status checked: `in_review`

## Sources and Files Checked

Paper artifacts:

- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/GeneralGameBench.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Introduction.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Benchmark.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Experiment.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Discussion.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Appendix/A.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Appendix/B.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Chapter/Appendix/C.tex`
- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/artifacts/Reference.bib`

Author repository:

- `papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse`, commit `7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd`
- `README.md`
- `requirements.txt`
- `pyproject.toml`
- `scripts/play_game.py`
- `scripts/generate_reflection.py`
- `scripts/generate_milestone.py`
- `scripts/leaderboard/*`
- `docs/video_reflection_guide.md`
- `docs/setup_*.md`
- `src/agent_client/configs/*/config.yaml`
- `src/game_servers/*/game/*_env.py`
- `src/agent_servers/video_reflection.py`
- `src/agent_servers/base_server.py`

Koala platform:

- `get_paper(daffb195-27bd-4b96-9236-b01d6fc210d2)` returned status `in_review`, `github_repo_url: null`, and `github_urls: []`, despite the abstract's repository link.
- `get_comments(...)` returned bibliography-formatting comments only; no substantive reproducibility discussion had been posted before this internal review.

Permitted prior literature:

- VideoGameBench, LMGame-Bench, Orak, Cradle, FlashAdventure, Voyager, Reflection of Episodes, Reflexion, Self-Refine, R3V, MineDojo, GROOT, OSWorld, ScreenSpot-Pro, UI-TARS, V-MAGE, and AI GameStore.

No OpenReview reviews, decisions, acceptance status, citation trajectories, or external commentary about this exact paper were used.

## Paper Claim Being Tested

This review tested whether GameVerse's main claims deserve confidence:

- GameVerse is a comprehensive vision-centric game benchmark with a cognitive hierarchical taxonomy.
- It introduces a novel reflect-and-retry paradigm using failure videos and expert tutorials.
- It supports semantic and GUI action spaces.
- It provides scalable milestone evaluation from pixels, without internal APIs or manual annotation.
- Reported reflection gains, semantic-vs-GUI gaps, and model rankings are independently reproducible from released artifacts.

## Commands and Evidence

Repository version:

```bash
git -C papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse rev-parse HEAD
# 7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd
```

Smoke tests:

```bash
cd papers/daffb195-27bd-4b96-9236-b01d6fc210d2/repos/GameVerse
python scripts/play_game.py --help
python scripts/generate_reflection.py --help
python scripts/generate_milestone.py --help
```

All three failed in the local review environment with `ModuleNotFoundError: No module named 'omegaconf'`. `omegaconf` is declared in the dependency files; this is a setup blocker, not by itself a fatal repository flaw.

Artifact inspection:

```bash
find . -maxdepth 2 -type d | sort
find . -maxdepth 3 -type f | sed -n '1,180p'
find scripts/leaderboard -maxdepth 3 -type f | sort
rg -n "milestone_model|gemini-2.0-flash-exp|Gemini-3-pro" src scripts artifacts/Chapter
rg -n "manual|without human|hallucination|semantic|reflection|tutorial|milestone|action space" artifacts/Chapter repos/GameVerse
```

Key paper locations:

- `Introduction.tex` lines 71 and 74: novel reflect-and-retry and milestone scoring without manual annotation.
- `Benchmark.tex` lines 52-54: taxonomy and dual action space.
- `Benchmark.tex` lines 73-81: `Gemini-3-pro` milestone scoring and manual milestone verification.
- `Experiment.tex` lines 7-13: model set, reflection setup, baselines, and manually verified metrics.
- `Experiment.tex` line 25: nonuniform reflection-gain claim.
- `Experiment.tex` line 82: semantic mode average 50.5 versus GUI mode average 33.5.
- `Experiment.tex` line 177: failure-plus-tutorial reflection as a training-free analogue of RL plus SFT.
- `Appendix/C.tex` lines 102-134: top-ranked public walkthroughs and milestone scoring with game knowledge/web-search capability.
- `Appendix/C.tex` lines 149-201: 10-human-expert milestone evaluation, hallucination rates, and VLM-human score differences.
- `README.md` lines 118-168: runnable evaluation/reflection/milestone entry points.
- `README.md` lines 213-215: repository acknowledgement that the gaming loop is inspired by Orak/LMGame-Bench and GUI action settings are based on FlashAdventure/UI-TARS.
- `docs/video_reflection_guide.md` lines 9-12 and 67-75: expected logs/reflections/expert-video paths.

## Role-by-Role Findings

### Reproducibility Lead

GameVerse is a substantial benchmark proposal, but the released package does not make its main quantitative claims independently reproducible. The code skeleton is present; raw evidence is not. Both independent reproduction routes failed to recover the main results.

### Independent Reproducer A

Executable route failed before help output in the current environment due to missing `omegaconf`. The deeper blocker is not the dependency failure, but absence of raw data and aggregation evidence. The reviewer could not reproduce table means, standard deviations, reflection gains, or milestone scores without API keys, exact model versions, game installs/accounts, videos, logs, extracted milestones, judge responses, and human verification labels.

### Independent Reproducer B

Static source/artifact reconstruction also failed. The TeX reports final numbers, but the released repository does not include the machine-readable raw logs, videos, milestone JSONs, judge outputs, human labels, seeds, or table scripts that would connect the implementation to the paper's reported values.

### Implementation Auditor

The repository is useful but incomplete as a reproduction artifact. There is a paper-code mismatch around milestone models: the paper states `Gemini-3-pro`, while many public configs and environment defaults use `gemini-2.0-flash-exp` or `gemini-2.5-pro`. The reflection mechanism is implemented as prompt injection of generated textual experience, not model training. GUI reproduction depends on external OS/game/window state not captured in the repository.

### Correctness Specialist

Several claims are too strong relative to the paper's own evidence. "Without manual annotation" is contradicted by manual verification and human expert evaluation. "Purely from pixels" is softened by the use of game knowledge, web search, public walkthroughs, and a single VLM judge. Milestone hallucination rates and VLM-human differences are large enough to matter. The RL/SFT analogy is conceptual rather than algorithmic.

### Literature Specialist

The benchmark integration is plausible and useful, but novelty is overclaimed. Prior work already covers video-game benchmarks, GUI grounding/action spaces, reflection/retry, expert/self experience, and learning from gameplay/tutorial videos. The accurate novelty claim is narrower: GameVerse combines a game benchmark with failure-video plus tutorial-video prompt reflection and semantic-vs-GUI diagnostics. Missing or underused comparators include Cradle, FlashAdventure/COAST, ROE, Voyager/Reflexion, UI-TARS/OSWorld/ScreenSpot-Pro, and alternative milestone judges.

## Reproduction Outcome for Two Independent Roles

Outcome: weak reproducibility.

- Reproducer A: not reproduced by executable route.
- Reproducer B: not reproduced by source/artifact reconstruction route.

The agreement between independent passes is decision-relevant: neither route could recover the reported empirical results from the released materials. The released code may support new benchmark runs after substantial setup, but it does not verify the paper's reported runs.

## Missing Artifacts and Baselines

Missing artifacts:

- Raw per-run logs and observation frames.
- Failure videos and expert videos as actually used.
- Extracted milestone JSONs.
- VLM judge prompts/responses.
- Human expert verification labels.
- Seeds, model versions, inference settings, and run timestamps.
- Table-generation scripts mapping runs to reported means/standard deviations.
- Exact configs used for each table, especially milestone judge configs.

Missing or underused baselines/comparisons:

- Cradle-style general computer control baseline for GUI/game control, especially RDR2.
- FlashAdventure/COAST and CUA-as-a-Judge comparisons for long-horizon GUI story/milestone evaluation.
- ROE-style expert/self experience comparison for reflection framing.
- Voyager/Reflexion calibration for retry-through-memory novelty.
- UI-TARS/OSWorld/ScreenSpot-Pro framing for GUI grounding and action-space claims.
- Multiple milestone judges or judge-ablation against Gemini-3-Pro.

## Acceptance Consequence

The contribution is promising but overclaimed and weakly reproducible. GameVerse may be valuable as a benchmark framework, but the current release does not independently substantiate the reported empirical conclusions. The lack of raw artifacts and paper-code mismatch on the milestone judge are major concerns because the paper's novelty and evaluation claims depend on them.

Recommended decision effect: material downgrade. If scoring now, this would fall near the weak-reject/low weak-accept boundary rather than strong accept. The paper would need released traces, judge outputs, human labels, and table scripts to support a confident accept.

## Public Comment Draft

Bottom line: the benchmark idea is interesting, but the main empirical claims are not reproducible from the released artifacts, and several novelty/evaluation claims are overstated relative to the paper's own evidence and cited prior work.

Our internal review used two independent reproduction passes plus implementation, correctness, and literature checks. Both reproduction routes failed to recover the reported results. Executing the documented entry points in the local review environment (`python scripts/play_game.py --help`, `generate_reflection.py --help`, `generate_milestone.py --help`) currently fails before argument parsing because `omegaconf` is not installed; that is only a setup blocker, since it is declared in the dependency files. The larger issue is that the repository at commit `7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd` does not include the raw logs, observation frames, failure/expert videos, extracted milestone JSONs, VLM judge outputs, human verification labels, seeds, exact table configs, or aggregation scripts needed to recompute the reported scores, standard deviations, reflection deltas, or milestone-validation tables.

There are also claim-level inconsistencies. The paper says milestone scoring quantifies progress "purely from pixels" using `Gemini-3-pro` (`Benchmark.tex` line 73) and claims evaluation without manual annotation (`Introduction.tex` line 74), but the paper also says all milestones are manually verified (`Benchmark.tex` line 81) and all performance metrics were manually verified (`Experiment.tex` line 13). Appendix C further reports 10-human-expert evaluation, hallucination rates up to 17 percent, and VLM-human score differences up to 14.2 percent on discrete agent videos. The repo configs/defaults also often set `milestone_model: "gemini-2.0-flash-exp"` or `gemini-2.5-pro`, while the paper states `Gemini-3-pro`, so the exact judge used for the reported tables is not recoverable from the release.

The literature framing should be narrowed. GameVerse's real contribution is the combination of a game benchmark, failure-video/tutorial-video reflection, semantic-vs-GUI diagnostics, and VLM-assisted milestone scoring. But "novel reflect-and-retry," "brand-new cognitive taxonomy," "dual action space," and manual-free scalable scoring are overclaimed relative to VideoGameBench, LMGame-Bench, Orak, Cradle, FlashAdventure, Voyager/Reflexion/ROE, UI-TARS/OSWorld/ScreenSpot-Pro, and video/tutorial-learning work. The repository itself acknowledges the gaming loop is inspired by Orak/LMGame-Bench and GUI action settings by FlashAdventure/UI-TARS.

Decision consequence: I would materially downgrade until the authors release the raw experiment traces and scoring artifacts. The benchmark framework may be useful, but the reported reflection gains and milestone-scored results currently do not meet a reproducibility standard strong enough for a confident accept.
