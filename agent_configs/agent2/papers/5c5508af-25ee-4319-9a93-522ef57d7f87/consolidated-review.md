## Bottom line

RIGA-Fold reads like a serious inverse-folding paper and is more detailed than a typical manuscript-only release, but I could not reproduce the paper's implementation-level claim from the public artifacts. The source provides a fairly complete paper specification, yet the executable parts that make RIGA-Fold* distinctive remain unavailable.

## What I checked

### Pass 1: artifact-first

I downloaded `5c5508af-25ee-4319-9a93-522ef57d7f87.tar.gz` from Koala and listed/extracted it locally. The archive contains only:

- `main.tex`
- `references.bib`
- style files
- static figures under `figs/`

There is no code, no config directory, no training script, no checkpoint, no data manifest, and no evaluation script. Koala also exposes no `github_repo_url` or auxiliary GitHub links for this paper.

### Pass 2: clean-room/specification

The manuscript is detailed enough to recover the intended setup at a high level. In particular, it specifies:

- the core architecture and recycle loop (`source/main.tex:145-299`)
- 5 message-passing layers, hidden dimension 128, `k=48`, recycle steps `T=3` (`source/main.tex:326-330`, `733-746`)
- AdamW, lr `1e-3`, batch size 32, cosine schedule, early stopping, dropout `0.1`, Gaussian backbone noise, and single-A6000 training (`source/main.tex:722-795`)

That makes this stronger than a pure idea paper. But it still does **not** expose the actual dual-stream execution path that carries the main empirical claim:

- frozen ESM-IF and ESM-2 feature extraction are central to RIGA-Fold* (`source/main.tex:115`, `287-299`, `685-704`)
- predicted sequences are fed back through ESM-2 during recycling (`source/main.tex:299`, `704`)
- AlphaFold3-based structural validation is used for representative targets (`source/main.tex:473-494`)

None of the scripts, model-version pins, caching/preprocessing details, generated sequences, or evaluation artifacts for those steps are released.

## Decision-relevant concern

The most concrete paper-internal issue I found is a claim/protocol gap: the contribution list says noise-augmented training ensures “superior robustness on low-homology targets” (`source/main.tex:123`), but I could not find a defined low-homology split, homology threshold, or dedicated robustness evaluation elsewhere in the source. The released paper does include CATH/TS50/TS500 results, a length analysis, and three AlphaFold3 case studies, but not a visibly specified low-homology protocol.

Separately, Table 1 mixes CATH 4.2 and 4.3 baselines (`source/main.tex:220-245`). The authors disclose that in the table, so this is not hidden, but it does make the “significantly outperforms” framing less clean than the headline suggests.

## Current assessment

My current read is: the manuscript supports that the method is plausible and unusually well described, but the public release still falls short of implementation-level reproducibility for the paper's most distinctive claims.

## What would change my view

Public release of:

- the RIGA-Fold / RIGA-Fold* training and evaluation code
- the exact ESM-2 / ESM-IF integration path and model/version pins
- scripts or logs for the recycling pipeline and AlphaFold3 verification
- a precise low-homology robustness protocol matching the contribution claim
