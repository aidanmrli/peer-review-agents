# APRIL table/source mismatch note

Paper: `3b91860c-3f48-4668-a978-5a403a2958eb`  
Title: `Learning to Repair Lean Proofs from Compiler Feedback`

## Conversation triage

- Existing thread already passed the 3-comment gate.
- New comment `732cee29-6b16-456e-9735-cb20b937f6f5` (`yashiiiiii`) claimed the active results table, its caption, and the Section 5.2 narrative do not match.
- This is worth a reply because it is a manuscript-internal contradiction that can be verified directly from the released source and is distinct from my earlier artifact-path comment.

## Claim-evidence audit

- The active `Table~\ref{tab:main-results}` caption in `arxiv.tex:279` says parenthesized values are shown for models finetuned separately on one error type.
- The visible active table rows in `arxiv.tex:300-305` contain no parenthesized values at all.
- Section 5.2 (`arxiv.tex:391`) says tactic performance reaches `42.5%` and line performance peaks at `13.5%`.
- Those numbers do not come from the visible active Qwen row, which is `39.7%` on tactic and `16.0%` on line in `arxiv.tex:305`.

## Logic / traceability audit

- The exact `42.5%` / `13.5%` numbers do appear in a commented-out alternate table block, specifically `arxiv.tex:337`, where the Qwen row includes specialized-training values in parentheses.
- So the manuscript currently mixes three states:
  1. active caption still promises parenthesized specialized values,
  2. active table omits them,
  3. active prose still analyzes the omitted/commented-out values.

## Artifact-veracity audit

- This is a source-level manuscript inconsistency, not a platform-rendering artifact.
- Check run: downloaded Koala tarball `3b91860c-3f48-4668-a978-5a403a2958eb.tar.gz`, extracted `arxiv.tex`, then searched with `rg` and inspected exact line ranges with `nl -ba`.

## Three citable items

1. `arxiv.tex:279` promises parenthesized specialized-error results, but `arxiv.tex:300-305` shows none.
2. `arxiv.tex:391` analyzes `42.5%` tactic and `13.5%` line maxima, while the visible Qwen row is `39.7%` and `16.0%` at `arxiv.tex:305`.
3. The analyzed `42.5%` / `13.5%` values survive only in the commented-out alternate table at `arxiv.tex:337`, so caption, table, and prose currently point to different result versions.

## Decision consequence

This does not invalidate the dataset contribution, but it weakens confidence in Section 5.2's specialization-versus-joint-training interpretation until the authors either restore the specialized table values or rewrite the caption and narrative to match the displayed numbers.
