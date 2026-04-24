# Independent Reproducer A Report

Paper: `13c5e02b-35fa-498b-8e7a-817f3e259d99`
Title: "UniDWM: Towards a Unified Driving World Model via Multifaceted Representation Learning"
Role: Independent Reproducer A
Date: 2026-04-24

## Claim Attempted

I attempted to reproduce the paper's central empirical claim from the paper text and official artifacts: UniDWM learns a multifaceted latent driving-world representation that substantially improves NAVSIM trajectory planning in the perception-label-free setting. The most decision-relevant reported result is Table 1 / `sec/3_method.tex` lines around the NAVSIM table: UniDWM with DINOv3-B reports `PDMS = 90.6`, outperforming label-free baselines including raw DINOv3-B (`85.4`), Epona (`86.2`), and World4Drive (`85.1`), and approaching the fully perception-supervised GaussianFusion result (`92.0`).

I also checked the secondary reproducibility-relevant claims available in the official source:

- 4D reconstruction on NAVSIM `navtest`: UniDWM reports Overall Chamfer `1.727` versus VGGT `3.000` and Spann3R `2.115`.
- Ablation: adding appearance, geometry, and dynamic generation to ego-pose reconstruction is reported to improve PDMS.
- Smoothness appendix: UniDWM is reported to produce smoother latent representations than the baseline on NAVSIM test split.

## Setup Used

Working directory:

```text
/home/mila/l/lia/peer-review-agents/agent_configs/agent2
```

Available official artifacts inspected:

```text
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/paper.pdf
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/0_abstract.tex
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/1_intro.tex
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/2_related.tex
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/3_method.tex
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/4_exp.tex
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/source.tar.gz
```

Environment observations:

```text
python: /home/mila/l/lia/peer-review-agents/.venv/bin/python
Python 3.12.12
pdflatex: /home/mila/l/lia/.TinyTeX/bin/x86_64-linux/pdflatex
latexmk: /home/mila/l/lia/.TinyTeX/bin/x86_64-linux/latexmk
pdftotext: not installed
```

I therefore used the LaTeX source as the primary paper text rather than extracting text from the PDF.

## Commands and Evidence

### Artifact inventory

Command:

```bash
rg --files papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts
tar -tzf papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/source.tar.gz | sort | sed -n '1,240p'
```

Observed result: the official archive contains only paper source, bibliography/style files, and figure PDFs. It does not contain training code, inference code, evaluation scripts, model checkpoints, config files, logs, seeds, NAVSIM preprocessing code, or a small dataset sample.

### Paper locations inspected

Commands:

```bash
sed -n '1,260p' papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/4_exp.tex
sed -n '1,260p' papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/3_method.tex
sed -n '390,450p' papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex
rg -n "Appendix|dataset|metrics|PDMS|NAVSIM|repo|github|code|availability|hyper|GRPO|seed|navtrain|navtest|UniDWM|DINOv3|DCAE" \
  papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex \
  papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec \
  papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.bib
```

Important source locations:

- `sec/4_exp.tex`, implementation details: DINOv3-B/DCAE static encoders, 12-layer 170M dynamic encoder, 459M next-frame DiT, 33M trajectory DiT, NAVSIM `navtrain`, `navtest`, image resize `224 x 384`, 50 epochs per stage, batch size 64, AdamW learning rate `1e-4`, weight decay `5e-2`, sampling steps 100 and 5.
- `sec/3_method.tex`, NAVSIM Table 1: label-free DINOv3-B baseline `PDMS = 85.4`, Epona `86.2`, World4Drive `85.1`, UniDWM DCAE `84.9`, UniDWM DINOv3-B `90.6`.
- `sec/4_exp.tex`, Table 2: 4D reconstruction `Overall = 1.727` for UniDWM.
- `sec/4_exp.tex`, Table 3 and Table 4: architecture ablation and GRPO finetuning.
- `main.tex`, Appendix "Dataset and Metrics": NAVSIM is described only at a high level, and PDMS is identified as the official metric.
- `main.tex`, Appendix smoothness analysis: metrics are described conceptually but no extraction script, feature files, or implementation details are supplied.

### Code artifact availability check

The task context states that cloning `https://github.com/Say2L/UniDWM` had already failed with public 404 / credential prompt. I independently checked the URL without using any forbidden future information about the paper.

Commands:

```bash
git ls-remote https://github.com/Say2L/UniDWM
curl -I -L --max-time 20 https://github.com/Say2L/UniDWM
```

Observed outputs:

```text
fatal: could not read Username for 'https://github.com': No such device or address
```

and:

```text
HTTP/2 404
server: github.com
```

Result: the linked GitHub repository is not publicly accessible from this environment. This blocks any code-based reproduction of the model, training recipe, preprocessing, inference, or NAVSIM evaluation.

### LaTeX source compilation check

Command:

```bash
latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex
```

Working directory:

```text
papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts
```

Observed result:

```text
! LaTeX Error: File `eso-pic.sty' not found.
!  ==> Fatal error occurred, no output PDF file produced!
```

This is not a scientific reproduction failure by itself because the prebuilt PDF is available, but it confirms that even source-level document reproduction is not fully self-contained in the local environment. Temporary build products from this failed check were removed.

### Arithmetic sanity checks

Because no runnable implementation is available, I performed the smallest meaningful independent checks available from the official source: verifying the table arithmetic used to support the empirical narrative.

Command:

```bash
python - <<'PY'
checks = {
    'PDMS gain UniDWM(DINOv3-B) vs DINOv3(B)': 90.6 - 85.4,
    'PDMS gain UniDWM(DINOv3-B) vs Epona': 90.6 - 86.2,
    'PDMS gain UniDWM(DINOv3-B) vs World4Drive': 90.6 - 85.1,
    'PDMS gap UniDWM(DINOv3-B) vs GaussianFusion supervised': 92.0 - 90.6,
    '4D recon overall reduction vs VGGT percent': (3.000 - 1.727) / 3.000 * 100,
    '4D recon overall reduction vs Spann3R percent': (2.115 - 1.727) / 2.115 * 100,
    'Ablation appearance increment from ego-only': 81.3 - 78.5,
    'Ablation geometry increment from ego-only': 80.0 - 78.5,
    'Ablation dynamic increment from ego-only': 80.9 - 78.5,
    'GRPO gain UniDWM': 84.9 - 82.4,
}
for k, v in checks.items():
    print(f'{k}: {v:.6g}')
PY
```

Observed output:

```text
PDMS gain UniDWM(DINOv3-B) vs DINOv3(B): 5.2
PDMS gain UniDWM(DINOv3-B) vs Epona: 4.4
PDMS gain UniDWM(DINOv3-B) vs World4Drive: 5.5
PDMS gap UniDWM(DINOv3-B) vs GaussianFusion supervised: 1.4
4D recon overall reduction vs VGGT percent: 42.4333
4D recon overall reduction vs Spann3R percent: 18.3452
Ablation appearance increment from ego-only: 2.8
Ablation geometry increment from ego-only: 1.5
Ablation dynamic increment from ego-only: 2.4
GRPO gain UniDWM: 2.5
```

Manual interpretation:

- The paper's stated `42.4%` 4D reconstruction improvement over VGGT is arithmetically consistent with Table 2.
- The ablation increments stated in prose for appearance (`+2.8`), geometry (`+1.5`), and dynamic generation (`+2.4`) are arithmetically consistent with Table 3.
- The strongest central PDMS comparison, UniDWM DINOv3-B over raw DINOv3-B, is a `+5.2` point improvement according to Table 1.
- These checks verify consistency of reported numbers, not their experimental validity.

## Reproduction Outcome

Outcome: blocked for the central empirical claim.

I could not independently reproduce the NAVSIM PDMS `90.6` result, the 4D reconstruction Chamfer `1.727`, or the smoothness metrics from the available artifacts. The blockers are concrete and material:

1. No runnable code is included in the official artifact archive.
2. The stated GitHub repository `https://github.com/Say2L/UniDWM` is not publicly accessible; GitHub returns HTTP 404 and `git ls-remote` cannot read refs without credentials.
3. No trained checkpoints are provided for UniDWM DINOv3-B, UniDWM DCAE, the baseline, or the GRPO variants.
4. No NAVSIM subset, preprocessing recipe, split manifest, dataloader, LiDAR projection code, or PDMS evaluation invocation is included.
5. The paper gives high-level training hyperparameters but not enough operational detail to reconstruct the experiment: no seeds, hardware/compute, optimizer schedule, augmentation settings, exact DINOv3/DCAE model identifiers, token dimensions, data filtering, checkpoint selection rule, evaluation command, or implementation of SIGReg in this architecture.
6. The official PDMS metric is named but not specified operationally in the artifact; without an evaluation script or predictions, the reported score cannot be recomputed.
7. The smoothness appendix lacks feature files or code for feature extraction, standardization, kNN construction, PCA ratio computation, or graph Laplacian smoothness.

The only reproducible unit from the provided artifacts is internal consistency of selected table arithmetic. That is a weak sanity check and does not reproduce the method.

## Match Assessment

Classification: blocked.

The reported numbers are arithmetically self-consistent where checked, but the central empirical claims are not independently reproducible from the paper and official artifacts. In this role, I would not assign strong reproducibility credit. At best, the paper provides enough textual detail to understand the claimed setup at a high level; it does not provide enough executable or inspectable material to verify that UniDWM actually achieves the claimed NAVSIM planning and 4D reconstruction results.

## Score Impact From This Role

The lack of accessible code, checkpoints, preprocessing, evaluation commands, and data manifests materially weakens confidence in the central claims. Because the paper's acceptance case depends heavily on empirical NAVSIM improvements and large-model training, failure to reproduce even a small executable unit should be a substantial negative under a reproducibility-first review standard. The arithmetic consistency checks prevent a stronger claim of internal numerical inconsistency, but they do not mitigate the missing-artifact blocker.
