# Independent Reproducer A Report

Paper ID: bd42a4b0-0fa8-448a-8d47-1d5b0ace30be
Title: Counterfactual Explanations for Hypergraph Neural Networks
Assigned role: Independent Reproducer A
Date: 2026-04-25 America/Toronto

## Task Scope

Attempt an artifact-first reproduction of the central empirical results from the paper text, official Koala artifacts, and the declared code link.

## Claim Attempted

Reproduce at least one row from the reported CF-HyperGNNExplainer tables, preferably Cora V1 accuracy/sparsity/size or the Cora runtime speedup, using the official code or documented commands.

## Setup Used

Working directory: `/home/mila/l/lia/peer-review-agents/agent_configs/agent2`
Relevant local artifact directory: `papers/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be/artifacts/`

Commands and checks:

```bash
curl -fsSL https://koala.science/storage/tarballs/bd42a4b0-0fa8-448a-8d47-1d5b0ace30be.tar.gz | tar -tzf -
curl -sS -o /dev/null -w '%{http_code} %{redirect_url}\n' https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716
curl -sS -o /dev/null -w '%{http_code}\n' https://anonymous.4open.science/api/repo/CF-HyperGNNExplainer-0716/file/
git ls-remote https://anonymous.4open.science/r/CF-HyperGNNExplainer-0716
python - <<'PY'
vals=[('V1 speedup',44.449/3.299),('V3 speedup',44.449/3.190),('Cora ours-CFGNN*',0.720-0.497),('Cora ours-CFGNN',0.720-0.499)]
for k,v in vals:
    print(f'{k}: {v:.6f}')
PY
```

## Evidence Examined

- `ExperimentalSetup.tex:4-18`: hardware, PyTorch Geometric, datasets, conversion, HGNN training details.
- `ExperimentalSetup.tex:39-71`: metrics and explainer hyperparameter search.
- `Results.tex:31-36`: main Cora/CiteSeer/PubMed results.
- `Results.tex:56-60`: Cora baseline comparison.
- `Results.tex:97-109`: runtime table.
- Declared code URL in `ExperimentalSetup.tex:6`.

## Observed Result

The source tarball is manuscript-only. It has no scripts, configs, Python files, notebooks, checkpoints, seeds, raw outputs, data manifests, result tables in machine-readable form, or executable instructions. The code URL redirects to an API endpoint returning HTTP 401, and it is not accessible through `git ls-remote`.

Only arithmetic checks can be reproduced:

- `44.449 / 3.299 = 13.473477`, matching the reported 13.5x V1 sparse speedup after rounding.
- `44.449 / 3.190 = 13.933856`, matching the reported 13.9x V3 sparse speedup after rounding.
- `0.720 - 0.497 = 0.223`, matching the Cora accuracy gap versus CF-GNNExplainer star-expanded.
- `0.720 - 0.499 = 0.221`, matching the Cora accuracy gap versus CF-GNNExplainer original graph.

## Reimplementation Feasibility

Blocked. A faithful run cannot be reconstructed from released materials. The first blocking item is the inaccessible implementation. Even ignoring the failed code link, the manuscript omits explainer optimization settings that materially affect the tables: beta, number of steps, initialization, restarts, stopping rule, V3 neighborhood radius, validation/model selection policy, and seeds.

## Match Status

Blocked. I recovered table arithmetic but did not reproduce any central empirical result.

## Concrete Failure Reason

The official artifacts do not provide runnable code or sufficient experimental specification. Since the metric is measured over trained HGNN predictions and optimized counterfactual masks, a reviewer cannot recover the results without the training/evaluation pipeline.

## Limitations

I did not attempt a speculative implementation because the missing settings are not incidental. Different reasonable choices for mask initialization, beta, stopping rule, thresholding, and graph preprocessing could materially change validity, sparsity, and explanation size.

## Confidence Level

High confidence that the official artifacts do not support artifact-level reproduction. Moderate confidence that a clean-room implementation would be underdetermined.

## Decision Impact

The central empirical claims should be treated as unreproduced. This is a substantial negative under the agent's reproducibility standard.
