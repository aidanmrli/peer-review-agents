# Implementation Auditor Report: GameVerse

Paper: `daffb195-27bd-4b96-9236-b01d6fc210d2`, "GameVerse: Can Vision-Language Models Learn from Video-based Reflection?"

Role: Implementation Auditor. This pass checked code, configs, artifacts, and paper-code alignment.

## Paper Claim Being Tested

The implementation audit tested whether the author-linked repository faithfully supports:

- The benchmark tasks and action modes described in the paper.
- The video-reflection workflow.
- The milestone scoring model and artifacts claimed in the paper.
- Reproduction of reported tables from released code and data.

## Sources and Files Checked

- `repos/GameVerse` at commit `7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd`
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
- Paper files `Benchmark.tex`, `Experiment.tex`, and `Appendix/C.tex`

## Commands and Observations

Version and repository inventory:

```bash
git rev-parse HEAD
# 7e95a4683a268ff3c1ccdd5bd19ff3563f8b7fcd

find . -maxdepth 2 -type d | sort
find . -maxdepth 3 -type f | sed -n '1,180p'
find scripts/leaderboard -maxdepth 3 -type f | sort
```

The repo contains source, configs, setup docs, leaderboard scripts, and one Slay the Spire communication mod jar. It does not include top-level raw `data/`, `logs/`, `outputs/`, or `results/` directories with the paper's evaluation evidence.

Smoke tests:

```bash
python scripts/play_game.py --help
python scripts/generate_reflection.py --help
python scripts/generate_milestone.py --help
```

All failed before CLI output with `ModuleNotFoundError: No module named 'omegaconf'` in the current environment. `omegaconf` is declared in `requirements.txt` and `pyproject.toml`, so this is an environment setup blocker rather than a code omission.

Milestone-model search:

```bash
rg -n "milestone_model|gemini-2.0-flash-exp|Gemini-3-pro" src scripts artifacts/Chapter
```

Findings:

- Paper: `Benchmark.tex` line 73 and `Appendix/C.tex` line 134 state that milestone scoring uses `Gemini-3-pro`.
- Script: `scripts/generate_milestone.py` lines 138-201 reads `env.milestone_model` from config and uses that model.
- Configs: many released configs use `milestone_model: "gemini-2.0-flash-exp"`, including `snake`, `tic_tac_toe`, `maze`, `metro`, `pvz`, `red_dead_redemption2`, `civilization`, `genshin`, `forza_horizon5`, and others. Some use `gemini-2.5-pro`.
- Environment classes likewise default many milestone models to `gemini-2.0-flash-exp`.

Reflection workflow search:

```bash
nl -ba docs/video_reflection_guide.md | sed -n '1,120p'
```

The guide requires logs, failure videos, `data/reflections/`, and `data/expert_videos/{game}/...`. These are operational instructions, but the paper's actual failure videos/reflections/expert-video artifacts are not included.

## Implementation Findings

1. The repository is a framework release, not a complete reproduction artifact.

It provides source and configs, but not the raw benchmark evidence needed to verify the paper's tables. The missing items include per-run logs, observation images, videos, milestone JSONs, VLM judge outputs, human verification labels, seeds, and table scripts.

2. There is a meaningful paper-code mismatch for the milestone judge model.

The paper repeatedly says `Gemini-3-pro` was used for milestone scoring. The released implementation is configurable, but many configs and defaults use `gemini-2.0-flash-exp` or `gemini-2.5-pro`, and the repo does not identify the exact configs used for the paper's tables. This matters because the paper's scalability and scoring claims depend on judge accuracy.

3. The reflection mechanism is implemented as prompt injection, not training.

`docs/video_reflection_guide.md` describes generating textual experience from failed and expert videos and saving it to `data/reflections/`. `src/agent_servers/base_server.py` injects `reflection_experience` into prompts when enabled. This matches a prompt-conditioning loop, but the paper's analogy to RL plus SFT should be treated as conceptual rather than algorithmic.

4. GUI-mode reproduction depends on external state not captured in the repo.

The benchmark spans local games, browser/GUI automation, and commercial games. Reproducing GUI results requires OS-specific input libraries, screen capture, window placement, installed games/accounts, calibration, and potentially real-time timing. The setup docs help, but exact experiment environments are not captured.

5. Platform metadata does not expose the repository link.

`get_paper` returned `github_repo_url: null` and `github_urls: []`, although the abstract mentions `https://github.com/THUSI-Lab/GameVerse`. The local clone exists and was inspected, but the platform metadata omission makes artifact discovery weaker for ordinary platform review.

## Reproduction Outcome

Outcome: implementation partially auditable but empirical results not reproduced.

The codebase plausibly implements the benchmark skeleton and reflection pipeline. It does not provide sufficient release artifacts to recompute the paper's reported benchmark results. The implementation cannot be treated as independent verification of the reported reflection gains.

## Exact Paper Locations

- `README.md` lines 118-168: evaluation and reflection/milestone scripts.
- `README.md` lines 213-215: gaming loop inspired by Orak/LMGame-Bench and GUI action based on FlashAdventure/UI-TARS.
- `Benchmark.tex` lines 73-81: milestone scoring and manual milestone verification.
- `Experiment.tex` lines 7-13: model setup, baselines, and manually verified metrics.
- `Appendix/C.tex` lines 134-201: milestone scoring model, human-expert validation, hallucination rates, and VLM-human matching differences.

## Literature References Used

Only implementation-relevant prior work mentioned by the repository/paper was used here: Orak, LMGame-Bench, FlashAdventure, UI-TARS, VideoGameBench, and Cradle.

## Final Synthesis and Score Impact

Implementation evidence is mixed. The code release is nontrivial and relevant, but it does not reproduce the paper's empirical claims. The milestone-model mismatch and lack of raw scoring artifacts are major concerns because the paper's key claims depend on judged progress and reflection gains. Score impact: major reproducibility downgrade.
