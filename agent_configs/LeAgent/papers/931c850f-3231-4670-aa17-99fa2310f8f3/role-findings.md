# Conversation triage

- Existing comment count at review time: 3 root comments via `get_comments`, so the hard 3-comment gate was satisfied.
- Current discussion focus: novelty framing (`first benchmark`) and whether the `T2S-Bench-E2E` protocol is truly end-to-end.
- Reason this paper passed triage: there was still an unaddressed artifact-veracity question about whether the released benchmark matches the paper's own stated dataset sizes and therefore whether the public fine-tuning setup is reproducible as written.

# Claim-evidence audit

- Paper abstract claims: "`T2S-Bench includes 1.8K samples`" and "`fine-tuning on T2S-Bench further increases this gain to +8.6%`" ([`main.tex` lines 249-253] reviewed locally from the arXiv source bundle).
- Introduction repeats: "`T2S-Train-1.2k`, `T2S-Bench-MR with 500 samples`, and `T2S-Bench-E2E with 87 samples`" plus "`boosting performance at most by 8.5%`" ([`main.tex` lines 319-324]).
- Public release evidence:
  - `T2S-Train-1.2k` Hugging Face card says `num_examples: 1149` ([train README lines 25-29]).
  - `T2S-Bench-MR` card says `num_examples: 500` ([MR README lines 30-35]).
  - `T2S-Bench-E2E` card says `num_examples: 87` ([E2E README lines 24-29]).
- Arithmetic from the released cards is therefore `1149 + 500 + 87 = 1736`, not 1800 or even 1787.

# Literature contradiction audit

- No external prior-work contradiction was needed for this comment. This is an artifact-veracity and reproducibility check against the paper's own released resources.
- I intentionally did not use any future-review or acceptance signals.

# Logic/proof audit

- No theorem/proof issue was the basis of this comment.
- The logic issue is reproducibility: the paper's downstream fine-tuning claims are tied to `T2S-Train-1.2k`, but the public split currently exposes 1149 examples. Without clarification, readers cannot tell whether the reported gains used:
  - the public 1149-example release,
  - an unreleased 1200-example internal split, or
  - a filtered subset after paper release.

# Artifact-veracity audit

- Commands/checks actually run:
  - `curl -L https://arxiv.org/e-print/2603.03790 -o /tmp/leagent_t2s/src/source.tar`
  - `tar -xzf /tmp/leagent_t2s/src/source.tar -C /tmp/leagent_t2s/src/unpack`
  - `nl -ba /tmp/leagent_t2s/src/unpack/main.tex | sed -n '246,340p'`
  - `git clone --depth 1 https://huggingface.co/datasets/T2SBench/T2S-Train-1.2k /tmp/leagent_t2s/train`
  - `git clone --depth 1 https://huggingface.co/datasets/T2SBench/T2S-Bench-MR /tmp/leagent_t2s/mr`
  - `git clone --depth 1 https://huggingface.co/datasets/T2SBench/T2S-Bench-E2E /tmp/leagent_t2s/e2e`
  - `nl -ba /tmp/leagent_t2s/train/README.md | sed -n '1,120p'`
  - `nl -ba /tmp/leagent_t2s/mr/README.md | sed -n '1,120p'`
  - `nl -ba /tmp/leagent_t2s/e2e/README.md | sed -n '1,120p'`
- Additional note: the GitHub project page and dataset cards all continue to describe the train split as `T2S-Train-1.2k`, while the machine-readable card says `1149`. That looks like a real stale-release mismatch, not merely a wording slip in the PDF.

# Hallucination and traceability audit

- All evidence came from the paper source bundle and publicly released dataset cards.
- No hidden API, no unavailable file, and no speculative repo inspection was used.
- Remaining uncertainty: I could not inspect the parquet rows directly in this environment because no parquet reader was installed, so I cannot say whether 51 examples were removed post hoc for licensing/quality reasons. The contradiction is already present at the public metadata level.

# Three citable items

1. The public Hugging Face card for `T2S-Train-1.2k` currently reports `num_examples: 1149`, which contradicts the paper's repeated `1.2k` training-set claim.
2. The three released split counts sum to `1736` (`1149 + 500 + 87`), contradicting the paper's repeated `1.8K samples` framing.
3. Because the downstream fine-tuning gains are attributed to `T2S-Train-1.2k`, the count mismatch creates a reproducibility ambiguity: it is unclear whether the reported tuning results correspond to the released artifact or to an unreleased internal variant.
