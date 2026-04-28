## DARC contradiction note

Bottom line: the manuscript appears to use **incompatible definitions of the "high-disagreement subset"**, so the headline claim that DARC works especially well on "genuinely controversial prompts" is not currently traceable to a single evaluation protocol.

### Evidence

1. `icml26.tex:2234-2247` defines the high-disagreement subset using **human disagreement**:
   - `D(s) := max_k sigma_human^2(s, y_k)`
   - top 20% prompts by `D(s)`.
2. `icml26.tex:2368-2369` later defines the high-disagreement subset as the top 20% prompts by **baseline proxy disagreement** `hat sigma`.
3. `icml26.tex:2461-2463` again defines the fixed high-variance subset via the **baseline risk proxy**.
4. `icml26.tex:915-919` interprets gains on this subset as performance on "genuinely controversial prompts."
5. `icml26.tex:3018-3144` separately documents large proxy-human mismatch cases, so proxy-ranked prompts are not interchangeable with human-high-disagreement prompts.

### Decision impact

This is narrower than a fatal bug, but it weakens one of the paper's main empirical takeaways. As written, the paper shows that DARC helps on prompts with high **proxy** disagreement and sometimes on buckets tied to **human** disagreement, but it does not keep those two evaluation regimes cleanly separated in the headline narrative. That makes the claimed connection to preference heterogeneity look stronger than the protocol actually establishes.

### Minimal fix

- Reserve "high-disagreement subset" for one definition only.
- Report proxy-ranked and human-ranked subset results under different names.
- Rewrite the main-text conclusion so proxy-ranked gains are not presented as direct evidence about human controversy unless the human-ranked result is the one being cited.
