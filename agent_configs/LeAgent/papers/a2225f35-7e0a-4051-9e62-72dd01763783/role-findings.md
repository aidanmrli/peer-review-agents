# COIN role findings

## Conversation triage

- Existing comment count before posting: 3.
- Current discussion already covers disambiguation risk, autoregressive-baseline uncertainty, and a missing prior-work concern.
- This paper passes the 3-comment gate, and the strongest remaining contradiction is novelty framing against an identifiable primary prior paper.

## Claim-evidence audit

- The submission repeatedly frames Context Reliance as newly "uncovered" in NTP-based unstructured editing:
  - `sections/1_introduction.tex:22`
  - `sections/3_preliminary_experiments.tex:12,30`
  - `sections/6_conclusion.tex:2`
- The paper's core delta is better supported as an NTP-specific diagnosis plus a distinct mitigation, not discovery of the broader phenomenon.

## Literature contradiction audit

- Primary prior paper checked: Haewon Park et al., `Context-Robust Knowledge Editing for Language Models`, arXiv:2505.23026v2 / ACL 2025 Findings.
- arXiv abstract states that current KE methods fail when preceding contexts are present and introduces CoRE to improve context robustness. That is materially the same problem class the submission presents as newly uncovered.
- Therefore the strongest novelty claim is overstated. The defensible novelty is narrower: theory and mitigation for NTP-based unstructured editing.

## Logic/proof audit

- No new proof bug established in this pass.
- The contradiction is framing-level: the empirical/theoretical story may still be valuable, but its novelty scope is mis-specified relative to prior work.

## Artifact-veracity audit

- Koala exposes no `github_urls`.
- No artifact contradiction was needed for this comment.

## Hallucination and traceability audit

- Repository-wide search of the released tarball found no `CoRE`, `Context-Robust`, or `Park` citation strings.
- This means the relevant prior work is not merely downplayed in prose; it appears absent from the submitted references.

## Three citable items

1. The paper repeatedly says it "uncovers" Context Reliance, but CoRE (Park et al., 2025) already identified preceding-context fragility in knowledge editing and proposed a dedicated robustness method.
2. The strongest defensible novelty is NTP-specific theory plus the COIN mitigation, not discovery of the broader failure mode.
3. The released tarball appears not to cite CoRE at all (`rg -n "CoRE|Context-Robust|Park"` returned no relevant bibliography hits), which weakens the novelty framing further.
