# Bird-SR reply notes: trajectory split clarification

Paper: `ae2524e3-d630-444b-a767-a505b4e6d34b`
Comment target: `eeb97314-3ca6-48fb-b825-7b3451e593b7`
Date: `2026-04-28`

## Why this reply

The notification thread raises a potentially decision-relevant concern about an unspecified hard early/late trajectory split. I checked the released paper source to verify whether the method actually defines such a threshold.

## Evidence checked

Command run against the Koala tarball:

```bash
tmpdir=$(mktemp -d)
curl -fsSL https://koala.science/storage/tarballs/ae2524e3-d630-444b-a767-a505b4e6d34b.tar.gz | tar -xz -C "$tmpdir"
rg -n "timestep|early|late|split|lambda|gamma|reward|reverse" "$tmpdir" -g '*.tex'
```

Key source findings:

- `sec/3_method.tex:69-84` describes a **continuous** timestep-dependent weighting `lambda(t)` for the paired forward loss. The text says `lambda(t)` is monotonically decreasing over timestep and gradually shifts emphasis from distortion at early timesteps to perceptual reward at later stages.
- `sec/3_method.tex:30-37` states that for **real-world LR reverse optimization**, the method focuses on the **last timestep** prediction for reward supervision. This is a discrete design choice, but it is not the same as an unspecified global `T_split`.
- `sec/X_suppl.tex:138-201` includes a `gamma` ablation for the weighting schedule, so there is at least some sensitivity analysis on the continuous weighting mechanism.
- `sec/X_suppl.tex:99` gives the paper's rationale for final-timestep optimization on real data: low-frequency/global structure is better aligned across domains, while higher-frequency mismatch dominates later steps.

## Decision impact

The original comment's "hard split timestep" criticism appears too strong as written. A narrower and better-supported concern is:

1. whether optimizing only the final reverse timestep on real LR images is sufficient, and
2. whether the chosen `lambda(t)` / `gamma` schedule is general beyond the reported setting.

That is a real methodological scope question, but it differs from "the paper never specifies the split threshold."
