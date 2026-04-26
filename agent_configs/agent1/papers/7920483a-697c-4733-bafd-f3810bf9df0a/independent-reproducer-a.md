# Independent Reproducer A Report

## Claim Tested

Could I reproduce the paper's main UVG/HEVC VOV rate-distortion curves and the claim that VOV achieves strong DISTS/FVD at extremely low bitrates from the released paper, source artifacts, and official code?

## Procedure

I started from the paper bundle and author-linked project page, not from the Koala metadata link alone.

Commands:

```bash
rg --files artifacts | rg '\.(py|ipynb|sh|yaml|yml|json|csv|tsv|txt|md|log|safetensors|ckpt|pt|pth)$'
curl -L -s https://compressionasadaptation.github.io/ | rg -n "github|Code|VisionAsAdaptations|href"
rg --files repos/VisionAsAdaptations | sed -n '1,220p'
sed -n '1,260p' repos/VisionAsAdaptations/README.md
sed -n '1,260p' repos/VisionAsAdaptations/video/README.md
sed -n '1,260p' repos/VisionAsAdaptations/video/train/README.md
sed -n '1,260p' repos/VisionAsAdaptations/video/scaling/README.md
sed -n '1,240p' repos/VisionAsAdaptations/video/eval/README.md
```

## Findings

The paper source bundle contains LaTeX and static figures, not runnable code or raw data. The only non-LaTeX data-like file under the paper bundle is `artifacts/00README.json`; the bundle has no Python scripts, shell scripts, configs, metric tables, logs, checkpoints, or bitstreams.

The official project page does link to a relevant implementation, `microsoft/VisionAsAdaptations`, and the local clone at commit `3d9091d52d49076459822a413a7b0cdd14d11b7e` contains plausible code for:

- image training/reconstruction under `image/`
- video two-stage training under `video/train/`
- scaling reconstruction under `video/scaling/`
- evaluation under `video/eval/`
- per-sequence video captions under `video/**/descriptions/`

However, the paper's central RD result is not reproducible end-to-end from these materials. The repo has no released `.safetensors`, `.pt`, `.pth`, `.ckpt`, `.json`, `.csv`, or `.log` experiment outputs in the searched paths. There are no raw RD metrics corresponding to the plotted UVG/HEVC figures and no decoded-frame outputs for the full benchmark set. The paper's Figures are static PNG/PDF files in the LaTeX bundle.

The paper states all benchmark videos are center-cropped/resized to 832x480 and evaluated on 81-frame clips (`artifacts/chapters/experiments.tex:30-33`), but the repo does not provide a dataset manifest specifying which exact 81-frame windows, source files, cropping coordinates, baseline encoded outputs, or baseline command lines were used.

## Reproduction Outcome

Blocked. I could identify plausible training and metric scripts, but I could not reproduce or independently recover the central rate-distortion curves without unavailable data preprocessing manifests, checkpoints/bitstreams, exact configs, baseline outputs, and raw result tables.

## Severity

High for acceptance. The paper is an empirical compression paper, and the core claim is comparative performance on UVG/HEVC. The release supports code inspection and possible future reruns, but not independent recovery of the reported main result.
