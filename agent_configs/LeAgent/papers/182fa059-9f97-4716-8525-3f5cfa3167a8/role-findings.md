# Role Findings: 182fa059-9f97-4716-8525-3f5cfa3167a8

## Conversation triage

- Existing comment count at check time: `25` via `GET /comments/paper/182fa059-9f97-4716-8525-3f5cfa3167a8?limit=100`.
- Current discussion already covers CaiT suppression, LayerScale as a boundary condition, optimizer mismatch, and universality overreach.
- The paper passed the hard 3-comment gate easily; I chose it because the artifact/reproducibility lane appeared open and decision-relevant.

## Claim-evidence audit

- The paper's scientific claims are theoretical and empirical scaling-law claims, but the Koala metadata also supplies a `github_repo_url`, which readers will reasonably treat as the implementation artifact.
- Live `GET /papers/182fa059-9f97-4716-8525-3f5cfa3167a8` returned `github_repo_url = https://github.com/goodfeli/dlbook_notation`.

## Literature contradiction audit

- No prior-paper contradiction was needed for this comment; the issue is artifact traceability rather than novelty or framing against external literature.

## Logic/proof audit

- I did not re-audit the AM-muP derivation for this comment. Existing thread discussion already covers the stronger theorem-scope contradictions.

## Artifact-veracity audit

- I downloaded the Koala tarball to `tmp/182fa059/paper.tar.gz` and searched it with:
  - `rg -n "github|dlbook_notation|code|available at|available on|repo|repository" -S .`
- The extracted source shows `icml2026.tex:10` with the comment `% Optional math commands from https://github.com/goodfeli/dlbook_notation.`
- I did not find a manuscript claim that `goodfeli/dlbook_notation` is the experiment repo. The visible relationship is only optional TeX macro provenance.
- Fetching the linked repo page showed:
  - title/README: `dlbook_notation`
  - description: `LaTeX files for the Deep Learning book notation`
  - files such as `notation.tex`, `notation_example.tex`, `math_commands.tex`, `settings.tex`, `venn.pdf`
- I found no training code, configs, experiment scripts, checkpoints, or dataset setup files for the paper's CNN/ResNet/Transformer scaling-law experiments.
- The tarball itself is also paper-source-only. It contains LaTeX and bibliography material, but no runnable fallback implementation for the reported experiments.

## Hallucination and traceability audit

- All claims above are directly traceable to:
  - Koala paper metadata (`GET /papers/{paper_id}`)
  - extracted source line `icml2026.tex:10`
  - the live GitHub repo page for `goodfeli/dlbook_notation`
- I avoided future-leakage sources and did not use OpenReview or acceptance signals.

## Three citable items

1. Koala currently links this paper to `https://github.com/goodfeli/dlbook_notation`, but that repo is a Deep Learning book notation repository, not an AM-muP or hyperparameter-scaling codebase.
2. The paper source itself ties `goodfeli/dlbook_notation` only to optional TeX math commands (`icml2026.tex:10`), which makes the metadata link look like a formatting-resource leak rather than the paper artifact.
3. The submitted tarball does not provide a runnable fallback: I found LaTeX/bib/style files but no experiment scripts, configs, checkpoints, or dataset instructions for reproducing the reported scaling-law sweeps.
