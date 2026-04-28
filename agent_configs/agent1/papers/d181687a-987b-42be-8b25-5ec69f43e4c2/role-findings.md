# R2-Router role findings

## Central claim and reproduction target

The paper claims `R2-Router` plus `R2-Bench` achieves comparable quality at `4-5x` lower cost by jointly selecting an LLM and output-length budget.

## Paper and artifact evidence checked

- Koala tarball source only; no runnable code or dataset artifact was present in the submission bundle.
- `main.tex:203` states the `4-5x` lower-cost headline and constructs `R2-Bench`, but the source-code/demo URL is commented out rather than active.
- `main.tex:358` says costs follow OpenRouter pricing.
- `main.tex:790` says the LLM pool uses OpenRouter per-token prices and explicitly notes that pricing is subject to change and was retrieved in January 2026.
- `main.tex:616` says `R2-Bench` can be reconstructed via GPU collection or API calls, but no manifest of raw responses, actual token counts, or frozen price table is included in the release.

## Reproducibility result from the smallest meaningful check actually run

I verified that the released artifact is manuscript-only and that the public code URL is commented out in the paper source. I also verified that the cost axis used for the main claim depends on mutable vendor pricing rather than a frozen cost snapshot bundled with the artifact.

## Implementation or correctness risks

- Without public code or `R2-Bench` records, an external reviewer cannot replay the router training/evaluation pipeline.
- Because pricing is acknowledged to change over time, the reported `QNC` / cost curves are not replayable from the manuscript alone.
- The paper discusses actual-length compliance, but the artifact does not include the per-query `(model, budget, actual_tokens, judged_quality, price_snapshot)` table needed to reconstruct the reported curves.

## Novelty/framing context

The curve-routing idea may still be valid; this is a reproducibility concern about the artifact supporting the quantitative cost-efficiency claim, not a claim that the method is unsound.

## Decision impact

This lowers my confidence in the `4-5x lower cost` headline and keeps my score below a strong-accept reading unless the authors release a replayable artifact or archived benchmark records.
