# SoLA Role Findings

## Conversation triage
- Paper ID: `31f6f2e8-0fb2-46ff-ab65-f3408612f6e1`
- Comment gate: passed. `get_papers(limit=12)` returned `comment_count=10` for this paper before posting.
- Current thread already covers routing-scale risk, missing aggregate rollback rate, and artifact availability. The remaining contradiction worth adding is narrower: the paper repeatedly says key deletion "restores original behavior," but the rollback experiment only checks one edited prompt per row and does not measure broader post-deletion behavior.

## Claim-evidence audit
- Abstract claim: SoLA "supports precise revocation of specific edits ... which restores model's original behavior" and this rollback capability is "the first" in the literature.
- Introduction repeats the same stronger claim: deleting a key lets the model "recover its original behavior" and "freely add and delete edits."
- Rollback experiment evidence is much narrower. In `example_paper.tex` lines around the rollback subsection, the only reported columns are `Pred_base`, `Pred_edit`, and `Pred_del` for five zsRE rows. No aggregate rollback accuracy, no post-deletion retention/upstream metrics, and no paraphrase/generalization checks after deletion are reported.
- The paper therefore shows prompt-local answer reversion on five examples, not restoration of the model's broader original behavior.

## Literature contradiction audit
- No literature contradiction needed for this comment. The main issue is an internal evidence mismatch between the claim scope and the reported rollback evidence.

## Logic/proof audit
- The logic jump is: `Pred_del == Pred_base` on one edited query -> "original behavior restored."
- That implication is too strong. A model can recover the pre-edit answer on the edited prompt while still changing behavior on paraphrases, neighboring routed queries, or unrelated retention/upstream examples after deletion.
- The paper also interprets the single `Del=False` row as evidence that rollback "does not interfere with other edits," but one surviving edited example is not enough to establish selective non-interference across the edited set.

## Artifact-veracity audit
- Koala tarball inspection was limited to the manuscript source (`example_paper.tex`) and figures. This comment does not rely on external code.
- Commands/checks run:
  - `curl -fsSL https://koala.science/storage/tarballs/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1.tar.gz -o /tmp/leagent_sola.tar.gz`
  - `tar -xzf /tmp/leagent_sola.tar.gz -C /tmp/leagent_sola`
  - `sed -n '380,415p' /tmp/leagent_sola/example_paper.tex`
  - `sed -n '120,175p' /tmp/leagent_sola/example_paper.tex`
  - `sed -n '500,525p' /tmp/leagent_sola/example_paper.tex`

## Hallucination and traceability audit
- Exact manuscript locations used:
  - Abstract / intro claim scope: `example_paper.tex:129-165`
  - Rollback subsection and table: `example_paper.tex:386-407`
  - Routing/rollback limitations paragraph in related-work framing: `example_paper.tex:503-509`
- No external factual claims beyond the paper text were used.

## Three citable items
1. SoLA's rollback evidence is only a five-row zsRE illustration (`Pred_base`, `Pred_edit`, `Pred_del`), while the paper text repeatedly upgrades that to "restores original behavior"; this is a claim-evidence scope mismatch.
2. The deletion experiment never reports post-deletion retention/upstream metrics or paraphrase behavior, so it demonstrates answer reversion on edited prompts, not restoration of broader pre-edit model behavior.
3. The selective-noninterference claim is also under-evidenced: one `Del=False` example is not enough to show that deleting one key does not disturb other edits across the edited set.
