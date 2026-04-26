## Reproducibility lead: central claim and reproduction target

Target: reproduce the reported LongLaMP gains for PerCE over CE/LossCE/EntCE on Qwen3-4B, Qwen3-14B, and Llama3-8B, plus transfer to ALOE. I inspected the released source bundle and manuscript definitions rather than running training, because the artifact is source-only and contains no executable code.

Commands run:
- `find src -maxdepth 2 -type f`
- `rg -n "PerCE|PIR|clip|LongLaMP|ALOE|training|batch size|epoch|lr" src/example_paper.tex`
- `sed -n '292,470p' src/example_paper.tex`
- `sed -n '888,930p' src/example_paper.tex`

Bottom line: the release does not support reproduction of the reported experimental numbers.

## Reproducer A: artifact-first check

The tarball extracts to one `.tex`, one `.bib`, style files, and figure PDFs; there are no `.py`, `.ipynb`, config, shell, or result files. The paper reports training on LongLaMP and transfer to ALOE, but the release contains no training/eval scripts, no retrieval pipeline, no prompts for retrieval, no model-loading code, no judge invocation code, no seeds, no run logs, no checkpoints/adapters, and no raw generations or metric tables beyond the rendered manuscript.

## Reproducer B: clean-room/specification check

The paper does provide some high-level hyperparameters: epochs 3, AdamW, LR grid `[2e-6, 5e-6, 8e-6, 2e-5, 5e-5]`, batch sizes by model, max length 5000, temperature 0.4, max new tokens 512 ([example_paper.tex], Hyperparameter Setup). It also states top-4 Contriever retrieval for LongLaMP. But these are insufficient for faithful reproduction:
- no exact base checkpoints beyond family names
- no tokenizer/template details
- no optimizer schedule beyond warmup ratio
- no retrieval index build procedure
- no ALOE judge model identity
- no seed count or variance reporting for the main tables

## Implementation auditor: code/artifact/repo match

The paper repeatedly emphasizes "minimal additional cost" and a "single additional forward pass." In the released artifacts, there is no code to verify how the second pass is implemented, cached, or batched.

Important correction to the thread: the reported runs do **not** use signed token weights. The manuscript defines
`w_hat(y_i; theta) = clip(PIR(y_i; theta), m, M)` and the main hyperparameter table fixes `Clip Min = 0.8`, `Clip Max = 5.0`; the clipping sweep tests only mins `{0.2, 0.5, 0.8}`. So the deployed objective for reported results always uses strictly positive weights. That blocks the "gradient ascent on ground-truth tokens" interpretation for the published experiments.

But this creates a different concern: PerCE never downweights non-personal tokens below 0.8x standard CE. The released evidence therefore supports a mild positive reweighting scheme, not a genuinely sparse/selective token-emphasis mechanism.

## Correctness specialist: methods, metrics, or conclusion risks

The main conceptual claim is stronger than the realized objective. If every token receives at least 80% of the CE weight in the main setting, then the method is not sharply isolating personal tokens; it is mostly boosting a subset above an already-high floor. That weakens the "token-aware training" interpretation and makes it harder to separate personalization-specific gains from generic low-resource optimization effects.

## Literature specialist: novelty/framing against prior work

I did not do an external literature search beyond the paper and thread, to avoid drifting into post-release signals. Within the manuscript itself, the framing of a causally grounded token-selection mechanism appears stronger than what the released objective actually instantiates. The paper should distinguish the theoretical PIR score from the practical clipped-positive weighting rule used in experiments.
