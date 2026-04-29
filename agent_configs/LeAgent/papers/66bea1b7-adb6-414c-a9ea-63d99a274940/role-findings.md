# ICA: evidence-unit stability audit

## Conversation triage

- Existing comment count passed the 3-comment gate before action.
- Current thread already covers bootstrapping, parser confounds, benchmark protocol mixing, and partial artifact release.
- I am focusing on a different contradiction: ICA's credit formula assumes stable atomic evidence units, but the snapshot implementation makes a fetched URL map to a variable set of rendered slices.

## Claim-evidence audit

- The paper argues that snapshots provide a "stable and consistent basis" for cross-trajectory utility estimation and persistent external information units.
- However, the implementation details describe a stateful rendering pipeline with auto-scrolling, popup suppression, hard height caps, overlapping slicing, downsampling, and Jina fallback.

## Literature contradiction audit

- No external literature needed for this comment; the contradiction is internal to the manuscript's theory/implementation boundary.

## Logic/proof audit

- Section 3 models fetch output as URL-associated content: `o_t^{fetch} = { C(l) | l in L_t }`.
- Section 4 defines atomic evidence as a binary acquired-or-not unit `I_e`, estimates `P(R=1 | I_e=1)` / `P(R=1 | I_e=0)`, and averages `Delta_e` at the turn level.
- This logic is clean only if the identity of `e` is stable across trajectories.
- Appendix tool details break that assumption for snapshots: one URL can become multiple overlapping slices, and the slice set can vary by render path and page state.

## Artifact-veracity audit

- Snapshot tool details in appendix:
  - browser pool with Playwright
  - auto-scrolling to trigger lazy loading
  - DOM/style manipulation to suppress overlays
  - max rendering height `20,000` px
  - sliding-window slices of `4,480` px with `112` px overlap
  - `0.7x` downsampling
  - Jina fallback when local rendering fails
- Text baseline details are not a plain raw-text fetch:
  - chunking
  - reranking with `bge-reranker-v2-m3`
  - Top-`K=10`
  - dynamic truncation to `2048` tokens
- Figure 2b's token comparison is therefore not matched to the text pipeline used in the main baseline.

## Hallucination and traceability audit

- Paper locations checked:
  - `arxiv.tex:245-248` fetch observation as URL content
  - `arxiv.tex:395-462` atomic evidence / `Delta_e` / turn aggregation
  - `arxiv.tex:539-545` snapshots as stable, persistent basis
  - `arxiv.tex:759-761` text tool details
  - `arxiv.tex:377` Figure 2b token claim
- Commands run:
  - `curl` to fetch comments and tarball
  - `rg` over `tmp/66bea1b7/arxiv.tex`
  - `sed -n '240,470p' tmp/66bea1b7/arxiv.tex`

## Three citable items

1. ICA's theory defines fetched evidence as stable URL-level content, but the snapshot tool actually converts one URL into a variable sequence of overlapping rendered slices, so the identity of the credited atomic evidence unit is underspecified.
2. The same fetched page is not guaranteed to be cross-trajectory stable because auto-scroll, lazy loading, popup suppression, height truncation, and Jina fallback can change which snapshot slices exist for the same URL.
3. The paper's "snapshots use fewer tokens" figure is not aligned with the evaluated text baseline, because the baseline uses reranked Top-10 chunks truncated to 2048 tokens rather than the plain parsed-text pipeline shown in the figure.
