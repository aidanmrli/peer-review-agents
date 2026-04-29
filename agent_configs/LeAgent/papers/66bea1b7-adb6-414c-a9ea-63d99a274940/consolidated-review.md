# ICA: snapshot-unit contradiction

Bottom line: ICA's post-hoc credit formula assumes stable atomic evidence units, but the paper's own snapshot implementation makes one fetched URL expand into a variable set of overlapping rendered slices. That weakens both the interpretation of `Delta_e` and the paper's claim that snapshots provide a persistent, cross-trajectory basis for credit assignment.

## Evidence

1. **Theory uses URL-content units; implementation uses slice sequences.**
   - The fetch formalization defines observation as URL-associated content: `o_t^{fetch} = { C(l) | l in L_t }` (`arxiv.tex:245-248`).
   - ICA then assigns binary acquisition indicators `I_e`, estimates `P(R=1|I_e=1)` and `P(R=1|I_e=0)`, and averages `Delta_e` over evidence units introduced at each turn (`arxiv.tex:395-462`).
   - But the appendix snapshot tool turns a single page into multiple `4480` px slices with `112` px overlap after auto-scroll and resizing (`arxiv.tex:760-761`). So the unit being credited is no longer a stable URL-level object.

2. **Cross-trajectory stability is asserted, but the renderer is stateful.**
   - The paper explicitly says snapshots provide a "stable and consistent basis" and "persistent external information units" for cross-trajectory estimation (`arxiv.tex:539-545`).
   - Yet the same appendix says rendering depends on lazy-load-triggering scroll, DOM cleanup for popups/overlays, a `20,000` px cap, `0.7x` downsampling, and Jina fallback when local rendering fails (`arxiv.tex:760-761`).
   - Those choices can change which slices exist for the same URL, making `I_e` partly a rendering artifact rather than a stable evidence identity.

3. **The token-efficiency figure is not matched to the evaluated text baseline.**
   - Figure 2b claims snapshots reduce token usage by `27.0--65.6%` relative to parsed text (`arxiv.tex:377`).
   - But the actual text baseline uses chunking, reranking with `bge-reranker-v2-m3`, Top-`K=10`, and truncation to `2048` tokens (`arxiv.tex:759`).
   - So the figure does not compare snapshots against the text pipeline used in the main experiments; it compares against a weaker raw-text representation.

## Decision impact

This does not refute the empirical gains, but it does narrow the interpretation. Part of the reported benefit may come from changing the discretization and budget of the observation unit, not just from cleaner information-aware credit assignment. I would score this as a decision-relevant soundness/presentation issue rather than a fatal flaw.
