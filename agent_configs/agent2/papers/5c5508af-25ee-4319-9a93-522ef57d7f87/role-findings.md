## Reproducibility lead: central claim and reproduction target
Target claim: RIGA-Fold and especially RIGA-Fold* improve inverse-folding recovery/perplexity on CATH 4.2, TS50, and TS500 through three interacting components: GAU, the Global Context Bridge, and a dual-PLM recycle loop using ESM-2 and ESM-IF (`source/main.tex:98-123`, `145-145`, `287-299`). Reproduction target is therefore not just the architecture sketch, but the full executable training/evaluation path for the geometric encoder plus frozen-PLM feature extraction and recycling.

## Reproducer A: artifact-first check
Koala tarball contents are manuscript-only: `main.tex`, `references.bib`, style files, and figures. There is no code, config directory, environment file, checkpoint, dataset manifest, or evaluation script in `paper.tar.gz`. The paper lists no `github_repo_url` and no `github_urls` on Koala. This means the central empirical claim is not directly runnable from public artifacts.

## Reproducer B: clean-room/specification check
The paper is unusually specific for a source-only release. It specifies 5 message-passing layers, hidden size 128, `k=48`, recycle steps `T=3`, AdamW with lr `1e-3`, cosine schedule, batch size 32, 100 epochs, dropout `0.1`, backbone noise `N(0, 0.02^2)`, RTX A6000 hardware, and standard CATH 4.2 split (`source/main.tex:326-330`, `722-795`). That is enough to understand the intended training setup, but still insufficient to reproduce the full pipeline because the exact implementation of the GAU/global-bridge modules, dual-stream fusion, frozen ESM-2/ESM-IF feature extraction/caching, and decoding/recycling path is absent.

## Implementation auditor: code/artifact/repo match
There is no public repo linked from the paper metadata despite repeated implementation-dependent claims. The manuscript says RIGA-Fold* feeds predicted sequences back through ESM-2 while keeping ESM-IF as a structural anchor (`source/main.tex:115`, `287-299`, `685-704`), but there is no executable release for the tokenizer/input formatting, batching, cache policy, or checkpoint versions of those frozen models. The AlphaFold3 structural-validity check is also paper-only (`source/main.tex:473-494`): no generated sequences, AF3 settings, or scripts are released.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
Two paper-internal risks matter for evaluation. First, the contribution bullet claims “superior robustness on low-homology targets” (`source/main.tex:123`), but I could not find any defined low-homology split, threshold, or dedicated robustness table/figure anywhere else in the source. Second, Table 1 explicitly mixes CATH 4.2 and 4.3 baselines (`source/main.tex:220-245`), so the strongest “significant” SOTA framing should be interpreted carefully even though the authors do flag version differences in the table note.

## Literature specialist: novelty/framing against permitted prior work
The framing is plausible: geometric message passing plus long-range/global routing plus PLM recycling is a meaningful systems combination for inverse folding. My main literature-facing concern is not missing prior art, but overclaim calibration. Given the mixed CATH-version table and absent executable artifact, the manuscript supports “promising, well-specified architecture with competitive numbers” more cleanly than “decisive, fully reproducible new SOTA.”
