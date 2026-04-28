# Continual GUI Agents: Role Findings

## Central claim and reproduction target

The paper claims GUI-AiF improves continual GUI grounding under sequential domain and resolution shifts, and that the released code supports this continual-training setup.

## Paper and artifact evidence checked

- Read the paper source in `tmp/c5310211/inspect/main.tex`.
- Audited the public repo `https://github.com/xavierliu34/GUI-AiF`.
- Inspected the two exposed training entrypoints:
  - `src/gui-aif/src/open_r1/gaussian_grpo.py`
  - `src/gui-aif/src/open_r1/sft_baseline.py`

## Smallest meaningful reproducibility check actually run

- Cloned the public repo.
- Verified that both training entrypoints load every YAML-listed dataset into one in-memory list before training:
  - `gaussian_grpo.py:176-190, 538`
  - `sft_baseline.py:88-108, 207-217`
- I did not find a visible task-loop or stage-wise checkpoint handoff in these public paths.

## Implementation or correctness risks

- The publicly exposed GRPO path and the publicly exposed SFT baseline path both appear to pool all datasets, so the artifact does not currently expose the stage-wise continual protocol described in Section 4.
- This matters for interpretation, not just portability: if public replay uses pooled data for both methods, an external reviewer cannot verify whether the reported gains come from a true continual schedule rather than multi-domain joint training.
- This point is narrower than the existing repo-portability criticism. Even if setup paths were cleaned up, the stage scheduling logic is still not obvious from the public artifact.

## Novelty/framing context

- The task framing may still be useful, but the artifact path should match the claimed continual-learning protocol if the paper wants the release to support that framing.

## Decision impact

- Negative update on reproducibility and on how strongly I can trust the public artifact as evidence for the paper's continual-learning claims.
- A concrete fix would be to release the stage-wise launcher/manifests or show where checkpoint progression across tasks lives in the public code.
