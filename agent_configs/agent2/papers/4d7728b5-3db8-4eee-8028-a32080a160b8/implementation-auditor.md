# Implementation Auditor Report

Paper: `4d7728b5-3db8-4eee-8028-a32080a160b8`, "Scalable Simulation-Based Model Inference with Test-Time Complexity Control"

Role: Implementation Auditor for agent2

Audit date: 2026-04-24

## Bottom Line

The released implementation does not support independent reproduction of the paper's central empirical claims. The paper states that code to reproduce results is available at `https://github.com/mackelab/prism` (`artifacts/source/main.tex`, line 642), but the provided `artifacts/prism` repository snapshot contains only `README.md`, whose entire substantive content is "Tobe published soon." The source archive is a LaTeX source package with pre-rendered figure PDFs, not executable code. No training, inference, evaluation, data-generation, environment, configuration, checkpoint, or figure-generation artifacts are present.

Severity for acceptance decision: high. Confidence: high.

## Artifact Inventory

Inspected paths:

- `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism`
- `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source`
- `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source.tar.gz`
- `papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/paper.pdf`

Findings:

- `artifacts/prism` is a shallow Git clone with remote `https://github.com/mackelab/prism`, commit `500742291609efeb5201902d5c1c5c41cda3d1e7` dated 2026-02-25, and only one tracked file: `README.md`.
- `artifacts/prism/README.md` is 30 bytes and says only:

```text
# prism

Tobe published soon.
```

- `artifacts/source.tar.gz` expands to LaTeX sources and figure PDFs only: `main.tex`, `appendix.tex`, `references.bib`, style files, `math_commands.tex`, `00README.json`, and pre-rendered PDFs in `figures/`.
- A search for executable or reproducibility files under `artifacts` found only `source/00README.json`; no `.py`, `.ipynb`, `.yaml`, `.yml`, `.toml`, `requirements*.txt`, `environment*.yml`, `Dockerfile`, `.sh`, `.ckpt`, `.pt`, `.npz`, `.npy`, or `.csv` files were present.
- The artifact tree is about 19 MB, dominated by the paper PDF, LaTeX source, and embedded figure PDFs.

Commands used:

```bash
find papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts -maxdepth 4 -type f | sort
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism remote -v
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism ls-tree -r --name-only HEAD
git -C papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/prism show --no-patch --pretty=fuller HEAD
tar -tzf papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts/source.tar.gz | sort
find papers/4d7728b5-3db8-4eee-8028-a32080a160b8/artifacts -type f \( -name '*.py' -o -name '*.ipynb' -o -name '*.yaml' -o -name '*.yml' -o -name '*.toml' -o -name '*.json' -o -name 'requirements*.txt' -o -name 'environment*.yml' -o -name 'Dockerfile' -o -name '*.sh' -o -name '*.ckpt' -o -name '*.pt' -o -name '*.npz' -o -name '*.npy' -o -name '*.csv' \) -printf '%p\n' | sort
```

## Code Paths Inspected

There are no implementation code paths to inspect in the released repository. The only repository path is:

- `artifacts/prism/README.md`

Paper/source paths inspected for claimed implementation details:

- `artifacts/source/main.tex`, especially lines 158-164, 199-202, 327, 342-368, 439-490, 499-508, and 641-642.
- `artifacts/source/appendix.tex`, especially lines 139-239, 260-284, 317-371, 440-450, 463-500, 599-622, 753-864, and 871-1120.
- `artifacts/source/00README.json`, which only describes the LaTeX build (`pdflatex`, TeX Live 2025).

## Paper-to-Code Matches

Only high-level textual matches can be checked:

- The paper claims use of JAX, Hydra, and utilities from `sbi` (`main.tex`, line 642). No corresponding dependency files, imports, configs, lockfiles, or package metadata are released.
- The paper describes a transformer encoder, model-posterior autoregressive decoder, diffusion parameter decoder, tokenizers, AdaLN conditioning, block attention masks, and EDM-style sampling (`main.tex`, lines 283-327; `appendix.tex`, lines 139-188). No model source code is released.
- The paper describes a streaming `SimulationDataset`, online data generation, RAdam, adaptive gradient clipping, EMA, ring buffer size `1e5`, and H100 training setup (`appendix.tex`, lines 220-232). No dataset class, training loop, optimizer config, or launch command is released.
- The paper lists symbolic and dMRI component formulae and priors in LaTeX (`appendix.tex`, lines 871-1120 and beyond), but there is no executable implementation to check whether these expressions, transforms, masks, priors, and constraints match the experiments.

## Paper-to-Code Discrepancies

1. Public code availability claim is false for the provided artifact snapshot.

   The paper says "Code to reproduce results is available at `https://github.com/mackelab/prism`" (`main.tex`, line 642). The official `artifacts/prism` clone has only `README.md` and no code. This is a direct paper-artifact mismatch.

2. No reproducible configurations despite explicit Hydra claim.

   The paper says Hydra was used to track configurations (`main.tex`, line 642), and it reports many configuration-sensitive settings: model dimensions, layer counts, batch sizes, optimizer, EMA, diffusion steps, acquisition protocol generation, and model priors (`appendix.tex`, lines 145-239, 833-864). No Hydra config tree or run logs are provided.

3. No implementation for core PRISM method.

   The central method depends on specific implementation choices: autoregressive model posterior, diffusion decoder, component tokenization, active-component masking, parameter bijections, and sampling solvers (`main.tex`, lines 286-327; `appendix.tex`, lines 153-188). Without code, none of these can be checked for correctness, shape behavior, masking semantics, or consistency with the written method.

4. No data generation or simulator implementation.

   The paper's empirical claims depend on online simulation for symbolic regression and dMRI (`main.tex`, line 327; `appendix.tex`, lines 220-232, 841-864). The source archive includes mathematical descriptions and tables but no simulator code, random seeds, exact typical UKB/HCP acquisition arrays, preprocessing, or generated samples.

5. No evaluation scripts for reported metrics.

   Claims about rRMSE, SBC calibration, rKSD, ESS, top-5 accuracy, evidence-estimate `R^2=0.97`, tractography correlation `0.96` versus `0.86`, and H100 runtimes cannot be reproduced because there are no metric scripts, plotting scripts, trained weights, logs, or raw metric files (`main.tex`, lines 353-368, 464-490, 505-508; `appendix.tex`, lines 260-284, 319-361, 463-500).

6. No trained checkpoints.

   The paper reports large training budgets: symbolic models trained for 24 hours and dMRI networks trained for 72 hours on H100 hardware (`main.tex`, line 365; `appendix.tex`, lines 232, 239). No checkpoints are released, so even inference-only verification of the figures is impossible.

7. No downstream neuroimaging pipeline artifacts.

   The paper claims XTRACT tractography comparisons against BedpostX and SBI baselines, plus Rumba/DTI/LOOCV comparisons on UKB/HCP data (`main.tex`, lines 481-490, 505-506; `appendix.tex`, lines 528-541, 599-622). No scripts, subject IDs, voxel selections, masks, preprocessing, command lines, or baseline configurations are included.

## Reproducibility Blockers

Critical blockers:

- Missing implementation repository: the official `prism` snapshot has no implementation.
- Missing environment: no package versions, requirements, lockfile, container, CUDA/JAX version, or installation instructions.
- Missing configs: no Hydra configs, hyperparameter files, run manifests, seeds, or trained-run metadata.
- Missing training scripts: no launch commands for symbolic, B3S, BSZT, or BSZT+conv experiments.
- Missing simulation code: no symbolic-regression simulator, dMRI simulator, spherical convolution implementation, noise models, prior transforms, or acquisition-generation code.
- Missing evaluation code: no SBC, rRMSE, rKSD, ESS, evidence-estimation, top-k classification, runtime, LOOCV, tractography, or plotting scripts.
- Missing checkpoints: no pretrained model parameters or EMA states.
- Missing data artifacts: no generated synthetic datasets, metric tables, figure source data, UKB/HCP subset identifiers, bval/bvec arrays, voxel lists, or tractography outputs.

Secondary blockers:

- The paper provides enough prose to understand intended methods, but not enough exact procedural detail to reconstruct the implementation independently. For example, the text specifies broad random acquisition generation but not the actual "typical" UKB/HCP bvals/bvecs arrays used in the mixture (`appendix.tex`, lines 841-847).
- Runtime claims depend on H100-specific batching and implementation efficiency, but no benchmark script or batch-size exactness beyond "30k voxels" is released (`main.tex`, line 490).
- Several results depend on random subspaces or random samples (e.g. 200-model symbolic subspace, 50-model dMRI subspaces, 100 posterior samples for model discovery), but no seeds or sampled model lists are released (`appendix.tex`, lines 275-284, 448-450, 615).

## Decision Impact

The implementation audit materially lowers confidence in the paper. The central acceptance case is empirical and implementation-dependent: PRISM is claimed to scale to combinatorial model spaces, support test-time complexity control, improve over SBMI and prior dMRI SBI pipelines, match evidence-based BMC, and run quickly on H100 hardware. None of those claims can be independently verified from the released implementation because there is no released implementation.

The LaTeX source contains detailed methodological descriptions and pre-rendered figures, which is useful for paper reading but not for artifact verification. As an implementation audit, this should be treated as weak reproducibility. If the paper's score depends on reproducible artifact support, this is a substantial negative: the repository contradicts the paper's software availability statement and blocks reproduction of all core empirical results.

## Confidence

High confidence in the artifact-availability finding. The repository snapshot, Git tree, source archive contents, and file-type search all agree that the official artifacts contain no executable implementation. Lower confidence on whether the authors have unreleased private code, but unreleased code does not support independent review.
