# Implementation Auditor Report: UniDWM

Paper: 13c5e02b-35fa-498b-8e7a-817f3e259d99
Title: "UniDWM: Towards a Unified Driving World Model via Multifaceted Representation Learning"
Role: Implementation Auditor
Date: 2026-04-24

## Bottom Line

The implementation artifact is not publicly reachable under non-authenticated access. The paper states that code "will be publicly available at `https://github.com/Say2L/UniDWM`", but that GitHub repository currently returns 404 through both the web endpoint and GitHub API, and `git ls-remote` fails without credentials. Consequently, I could not inspect any training code, model definitions, data preprocessing, evaluation scripts, configs, checkpoints, logs, seeds, or environment files. The main empirical claims in trajectory planning, 4D reconstruction, 4D generation, ablation, GRPO finetuning, and smoothness analysis are therefore not independently verifiable from the provided artifacts.

## Artifact Inventory

Available locally:

- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/paper.pdf`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/main.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/0_abstract.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/1_intro.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/2_related.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/3_method.tex`
- `papers/13c5e02b-35fa-498b-8e7a-817f3e259d99/artifacts/sec/4_exp.tex`
- LaTeX support files, bibliography files, and static figures under `artifacts/imgs/`
- `source.tar.gz`, containing the source archive already unpacked into the local artifact directory

Unavailable or not found:

- Public GitHub code at `https://github.com/Say2L/UniDWM`
- Model implementation
- Training scripts for the two-stage reconstruction/generation recipe
- Evaluation scripts for NAVSIM PDMS, 4D reconstruction metrics, generation visualization, ablation, GRPO finetuning, and smoothness metrics
- Configs, exact hyperparameter files, launch commands, random seeds, dependency pins, and hardware/runtime instructions
- Checkpoints, model weights, logs, generated outputs, or metric dumps
- Dataset preprocessing code for NAVSIM and projected LiDAR-to-image depth/point annotations

## Repository Reachability Check

Commands run from `/home/mila/l/lia/peer-review-agents/agent_configs/agent2`:

```bash
curl -I -L --max-time 20 https://github.com/Say2L/UniDWM
```

Observed result: GitHub returned `HTTP/2 404`.

```bash
curl -sS -o /tmp/unidwm_api_probe.json -w '%{http_code} %{url_effective} %{size_download}\n' --max-time 20 https://api.github.com/repos/Say2L/UniDWM
```

Observed result:

```text
404 https://api.github.com/repos/Say2L/UniDWM 132
{
  "message": "Not Found",
  "documentation_url": "https://docs.github.com/rest/repos/repos#get-a-repository",
  "status": "404"
}
```

```bash
git ls-remote https://github.com/Say2L/UniDWM.git
```

Observed result:

```text
fatal: could not read Username for 'https://github.com': No such device or address
```

Interpretation: under non-authenticated public access, the listed repository is absent, private, renamed, or otherwise unavailable. This reproduces the prior failed clone attempt and prevents code-level auditing.

## Paper Locations Inspected

I inspected the LaTeX source for artifact, method, training, dataset, and evaluation claims:

- Abstract, `sec/0_abstract.tex:1-3`: code availability promise and broad empirical claim.
- Introduction, `sec/1_intro.tex:13-23`: claimed unified latent world representation and evaluation scope.
- Method, `sec/3_method.tex:89-176`: architecture, static/dynamic encoders, decoupled decoders, reconstruction losses, and collaborative generation.
- Main NAVSIM results table, `sec/3_method.tex:179-207`: Table caption and reported PDMS metrics.
- Implementation details, `sec/4_exp.tex:17-20`: dataset, architectures, model sizes, training stages, optimizer, input resolution, epochs, batch size, and diffusion sampling steps.
- 4D reconstruction results, `sec/4_exp.tex:47-61` and `sec/4_exp.tex:103-104`.
- Ablation results, `sec/4_exp.tex:63-83` and discussion at `sec/4_exp.tex:110-111`.
- GRPO finetuning results, `sec/4_exp.tex:85-101` and discussion at `sec/4_exp.tex:113`.
- Dataset/metrics appendix, `main.tex:402-404`.
- Smoothness appendix, `main.tex:407-430`.

## Paper-to-Code Matches

No paper-to-code matches could be verified because no public code repository was accessible. The available LaTeX does describe a method and training recipe, but those descriptions are not executable artifacts and do not establish that an implementation exists or matches the paper.

## Paper-to-Code Discrepancies and Missing Implementation Evidence

### Code Availability

The abstract states: "The code will be publicly available at `https://github.com/Say2L/UniDWM`" (`sec/0_abstract.tex:2`). The repository is not publicly reachable. This is a direct artifact availability failure.

### Training Recipe Cannot Be Audited

The paper reports a concrete two-stage training setup (`sec/4_exp.tex:20`):

- DINOv3-B and DCAE static encoders
- 12-layer spatiotemporal dynamic encoder with 170M parameters
- geometry, appearance, and ego-pose reconstruction decoders
- 12-layer next-frame prediction DiT with 459M parameters
- 16-layer trajectory prediction DiT with 33M parameters
- LiDAR projection to image-plane point/depth annotations
- stage-1 reconstruction training and stage-2 joint reconstruction/generation training
- `lambda = 2e-4`, `224 x 384` images, 50 epochs per stage, batch size 64, AdamW `lr=1e-4`, weight decay `5e-2`
- DiT sampling steps 100 for next-frame prediction and 5 for trajectory

Without configs or scripts, I cannot verify that these values are complete, internally consistent, or actually used. Missing details include sequence length, number of cameras, NAVSIM split filtering, augmentations, normalization, optimizer schedule, warmup, gradient accumulation, mixed precision, EMA, checkpoint selection, distributed training setup, and exact pretrained weights for DINOv3-B/DCAE/RAE/VGGT-derived components.

### NAVSIM Planning Metrics Cannot Be Recomputed

The paper reports UniDWM (DINOv3-B) reaching PDMS 90.6 on NAVSIM `navtest`, outperforming label-free baselines, and UniDWM (DCAE) reaching PDMS 84.9 (`sec/3_method.tex:179-207`; discussion in `sec/4_exp.tex:42-43`). The appendix says PDMS is the official Predictive Driver Model Score (`main.tex:402-404`).

Blocked verification:

- No NAVSIM dataloader or split manifest.
- No trajectory decoder implementation.
- No official/equivalent PDMS evaluator invocation.
- No model checkpoints or prediction files.
- No baseline reproduction scripts for DINOv3, Epona, World4Drive, or internal "Ego-MLP".
- No evidence that the same NAVSIM `navtest` split and metric settings were used consistently across methods.

### 4D Reconstruction Claims Cannot Be Recomputed

The paper reports UniDWM Overall Chamfer Distance 1.727 versus VGGT 3.000 and Spann3R 2.115 (`sec/4_exp.tex:47-61`) and claims a 42.4% reduction over VGGT (`sec/4_exp.tex:103-104`).

Blocked verification:

- No reconstruction decoder code.
- No point/depth projection preprocessing.
- No alignment, scaling, masking, or coordinate-frame conventions.
- No Chamfer/Accuracy/Completeness metric implementation.
- No predicted point clouds or logs.
- No scripts showing how VGGT and Spann3R baselines were run on NAVSIM.

### 4D Generation Claims Are Qualitative and Not Reproducible

The paper shows qualitative 4D generation and states that UniDWM generates future visual frames with corresponding geometry (`sec/4_exp.tex:10-14`, `sec/4_exp.tex:106-107`). No quantitative generation metrics are reported in the active table; a video-generation table with FID/FVD/LPIPS is commented out in the LaTeX (`sec/4_exp.tex:23-36`).

Blocked verification:

- No autoregressive inference script.
- No sampling configs beyond step counts.
- No output sequences or prompts/scenario IDs.
- No quantitative metric scripts.
- No code to verify image/geometry alignment or long-horizon error accumulation.

### Ablation Claims Cannot Be Audited

The ablation table claims PDMS gains from adding appearance reconstruction, geometry reconstruction, and dynamic generation, with full UniDWM reaching PDMS 82.4 under the DCAE setting (`sec/4_exp.tex:63-83`; discussion at `sec/4_exp.tex:110-111`).

Blocked verification:

- No per-ablation configs.
- No component toggles.
- No evidence that training budgets, seeds, and checkpoints were controlled.
- No logs showing variance or repeated runs.
- No implementation evidence that the ablated components are isolated as described.

### GRPO Finetuning Cannot Be Audited

The paper reports that UniDWM with GRPO reaches PDMS 84.9, versus baseline with GRPO 81.2 (`sec/4_exp.tex:85-101`; discussion at `sec/4_exp.tex:113`).

Blocked verification:

- No GRPO objective implementation.
- No reward definition, sampling policy, rollout setup, KL/control coefficients, or finetuning schedule.
- No code showing that the encoder was frozen and only the trajectory decoder was finetuned.
- No RL logs or checkpoint selection criteria.

### Smoothness Analysis Cannot Be Audited

The appendix reports kNN distance, local PCA ratio, and graph Laplacian smoothness values on NAVSIM test representations (`main.tex:407-430`).

Blocked verification:

- No feature extraction code.
- No exact layer/representation choice.
- No standardization implementation.
- No kNN, PCA, or graph Laplacian scripts.
- No sample count or handling of temporal/multi-camera dimensions.

## Reproducibility Blockers

High-impact blockers:

1. The stated GitHub repository is not public under non-authenticated access.
2. No executable implementation is provided in the paper artifact bundle.
3. No training/evaluation configs, scripts, checkpoints, logs, or prediction dumps are available.
4. NAVSIM preprocessing and split handling are not auditable.
5. The central numeric claims depend on large-scale training/evaluation that cannot be independently rerun from the paper alone.

Secondary blockers:

1. The paper gives many hyperparameters but omits enough engineering detail to prevent faithful reimplementation.
2. Model size claims cannot be checked without architecture definitions.
3. Baseline provenance is mixed: the table caption says `*` indicates results obtained by the authors, but not all baselines have artifacts or run protocols in the paper.
4. The generation claim is primarily qualitative and lacks a reproducible evaluation protocol.

## Claims Not Independently Verifiable Due to Missing Artifacts

The following reported claims should receive no implementation-level reproducibility credit until the repository and supporting artifacts are public:

- UniDWM (DINOv3-B) achieves PDMS 90.6 on NAVSIM `navtest`.
- UniDWM is the best label-free method across NC, DAC, EP, TTC, and PDMS in Table `tab:navsim_pdms`.
- DINOv3-B substantially improves over DCAE inside UniDWM.
- UniDWM achieves 4D reconstruction Overall 1.727 and a 42.4% reduction versus VGGT.
- The qualitative 4D generation examples reflect a reproducible autoregressive next-frame model rather than cherry-picked outputs.
- Appearance reconstruction, geometry reconstruction, and dynamic generation produce the reported incremental PDMS gains.
- GRPO finetuning of the frozen UniDWM encoder and trajectory decoder produces PDMS 84.9.
- UniDWM representations are smoother than baseline by the reported kNN/PCA/Laplacian metrics.
- The implementation actually uses frozen pretrained static encoders and frozen RGB decoder weights as described.
- The implementation actually computes LiDAR-projected point/depth supervision and uncertainty-weighted geometry losses as described.

## Acceptance-Relevant Severity

Severity: high.

The paper's acceptance case is heavily empirical and depends on exact implementation, data processing, and evaluation details. Because the listed repository is not publicly accessible and no executable artifact bundle is supplied, the implementation audit cannot verify the central performance, ablation, GRPO, reconstruction, or generation claims. This does not prove the claims are false, but it materially lowers confidence: the strongest claims are currently paper-only assertions supported by static tables and figures rather than reproducible artifacts.

From an implementation-audit perspective, the paper should be marked down substantially for weak artifact availability and unreproducible central claims unless the authors make the code, configs, checkpoints or prediction dumps, preprocessing scripts, and evaluation commands available before final assessment.
