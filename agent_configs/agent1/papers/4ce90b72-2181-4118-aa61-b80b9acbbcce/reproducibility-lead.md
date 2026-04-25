# Reproducibility Lead Report

Paper ID: `4ce90b72-2181-4118-aa61-b80b9acbbcce`

Title: `Delta-Crosscoder: Robust Crosscoder Model Diffing in Narrow Fine-Tuning Regimes`

Assigned role: Reproducibility Lead

Date: 2026-04-24

## Task Scope

I coordinated the internal review for `agent1` and checked whether the paper's main claims can be independently reproduced from the submitted paper, Koala artifacts, and permitted prior literature.

The central claim tested is that Delta-Crosscoder reliably recovers causally relevant fine-tuning-induced latents across 10 narrow fine-tuning model organisms, supports steering and partial mitigation, outperforms SAE-based crosscoder baselines, and approximately matches non-SAE ADL-style model diffing.

## Evidence Examined

Koala metadata:

- `github_repo_url: null`
- `github_urls: []`
- `pdf_url: /storage/pdfs/4ce90b72-2181-4118-aa61-b80b9acbbcce.pdf`
- `tarball_url: /storage/tarballs/4ce90b72-2181-4118-aa61-b80b9acbbcce.tar.gz`
- `status: in_review`

Local artifacts:

- `artifacts/example_paper.tex`
- `artifacts/example_paper.bib`
- `artifacts/paper.pdf`
- `artifacts/source.tar.gz`
- five static figure PDFs under `artifacts/figures/`
- style files and `00README.json`

Role reports:

- `independent-reproducer-a.md`
- `independent-reproducer-b.md`
- `implementation-auditor.md`
- `correctness-specialist.md`
- `literature-specialist.md`

## Commands and Checks

Representative commands:

```bash
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -maxdepth 4 -type f | sort
tar -tzf papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/source.tar.gz | sort
find papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts -type f \
  \( -name '*.py' -o -name '*.ipynb' -o -name '*.sh' -o -name '*.yaml' \
     -o -name '*.yml' -o -name '*.csv' -o -name '*.jsonl' -o -name '*.pt' \
     -o -name '*.safetensors' -o -name '*.npz' -o -name '*.pkl' \) -print | sort
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '273,393p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '505,560p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '745,831p'
nl -ba papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex | sed -n '882,1158p'
```

Small arithmetic/accounting check:

```bash
python3 - <<'PY'
import re, pathlib
tex = pathlib.Path('papers/4ce90b72-2181-4118-aa61-b80b9acbbcce/artifacts/example_paper.tex').read_text()
rows, org = [], None
for line in tex.splitlines():
    m = re.search(r'\\multirow\{4\}\{\*\}\{([^}]*)\}', line)
    if m:
        org = m.group(1)
    m = re.search(r'&\s*(Delta|TopK-200|TopK-400|DSF)\s*&\s*(\d+)\s*&\s*([0-9.]+)\s*&\s*(\d+)\s*&\s*([0-9.]+)', line)
    if m and org:
        rows.append((org, m.group(1), int(m.group(2)), float(m.group(3)), int(m.group(4)), float(m.group(5))))
print('organisms', len(set(r[0] for r in rows)), 'rows', len(rows))
for org in sorted(set(r[0] for r in rows)):
    rs = [r for r in rows if r[0] == org]
    delta = [r for r in rs if r[1] == 'Delta'][0]
    best_dead = min(rs, key=lambda r: r[5])
    best_ev = max(rs, key=lambda r: r[3])
    print(org, 'Delta EV', delta[3], 'best EV', best_ev[1], best_ev[3], 'Delta dead%', delta[5], 'best dead', best_dead[1], best_dead[5])
PY
```

Observed:

- The tarball is LaTeX and static figures only.
- The code/config/data/checkpoint search found only `00README.json`.
- The appendix metric table contains 8 organism rows, while the paper repeatedly claims 10 organisms.
- Delta has the lowest dead-feature rate in only 3 of the 8 tabled organisms and the highest explained variance in only 1 of 8.

## Role-by-Role Findings

### Independent Reproducer A

Reproducer A found that central reproduction is blocked. The artifact bundle has no implementation, data, configs, checkpoints, logs, raw figure data, latent dictionaries, activation caches, or raw generations. The only successful check was a minor appendix arithmetic check: 29 of 32 dead-feature percentages match the reported counts and dictionary sizes, with three small one-decimal discrepancies. This validates reporting arithmetic, not the empirical claims.

### Independent Reproducer B

Reproducer B independently reached the same conclusion. The paper is not reproducible from the released artifacts, and a faithful clean-room implementation is materially under-specified. Reproducer B also identified two important consistency issues: the appendix metrics table reports only 8 organism rows despite repeated 10-organism claims, and the appendix reports a relative decoder norm of `52.5` even though the stated formula is a ratio bounded in `[0, 1]`.

### Implementation Auditor

The implementation audit found no GitHub repository in Koala metadata and no implementation files in the tarball. The submitted materials contain only TeX source, bibliography/style files, and static figure PDFs. There is no Delta-Crosscoder code, baseline code, environment file, model checkpoint identifier list, finetuned adapter, data manifest, activation cache, learned dictionary, raw steering output, grader prompt/output, metric script, figure data, or hardware/runtime log. This is a high-severity implementation audit failure for an implementation-heavy interpretability paper.

### Correctness Specialist

The correctness pass found multiple decision-relevant issues:

1. The first delta-loss definition is equivalent to penalizing the difference between base and fine-tuned reconstruction residuals when the same sparse code is used; it does not by itself establish a new fine-tuning direction.
2. The paper says `Delta = b-a` need not use matched inputs, but unmatched activations confound model-induced differences with content differences.
3. Relative decoder norm is defined as `||d_base||/(||d_base||+||d_ft||)`, where large values are base-specific, yet the evaluation repeatedly selects the "right tail" for fine-tuning-induced latents.
4. The appendix's reported `52.5` relative decoder norm is impossible under that bounded ratio.
5. The Dual-K/BatchTopK objective is under-specified, including the actual non-shared sparsity budget, the relation between `alpha`, base sparsity, shared multiplier, and AuxK, and whether the sparsity term is a true loss or an implicit constraint.
6. Steering shows intervention effects but does not by itself prove the selected latents are necessary causes of natural model behavior.
7. The "false positive" discussion actually defines method-level failure to recover a relevant latent, which is a false negative or coverage failure, not a false positive.
8. The appendix metric table contradicts the prose claim that Delta-Crosscoder maintains comparable reconstruction and similar or lower dead-feature rates across most settings.
9. The ADL comparison is not a matched evaluation because Delta-Crosscoder artifacts are graded with a separate `GPT-5.2` grader and compared to best reported ADL task performance from another protocol.

### Literature Specialist

The literature pass found a plausible narrower contribution but overbroad framing. Delta-Crosscoder is best described as an integration of BatchTopK sparse dictionaries, crosscoder model diffing, dedicated/shared feature partitioning, activation-difference supervision, contrastive activation data, and residual-stream steering. Important neighboring work includes crosscoders, DSF/DFC, BatchTopK, ADL/static activation differences, model diff amplification, persona features/vectors, refusal directions, Representation Engineering, ActAdd, CAA, and CAFT. Missing or underused baselines include same-architecture DFC-style partitions without the delta loss, BatchTopK plus Latent Scaling diagnostics, static ADL/mean activation-difference baselines, persona/refusal-vector baselines, and mitigation baselines such as CAFT.

## Reproduction Outcome

Central empirical reproduction: blocked.

What was reproduced or checked:

- Artifact inventory and absence of implementation artifacts.
- Dead-feature table arithmetic, mostly matching.
- Internal consistency checks on the metric table and relative decoder norm definition.
- Paper-level method readability at a high level.

What was not reproduced:

- Delta-Crosscoder training.
- Learned sparse dictionaries or latent rankings.
- Causal steering/ablation results.
- The null experiment.
- The no-finetuning-data and dictionary-size ablations.
- Baseline training and coverage claims.
- GPT-5.2 grader comparison to ADL.
- Any raw figure data, prompt-level outputs, or confidence estimates.

Overall classification: weak reproducibility for the central claims.

## Score Impact

The idea is interesting and may be useful, but the acceptance case is dominated by broad implementation-sensitive empirical claims that cannot be verified from the submitted materials. The correctness and metric-accounting issues further reduce confidence in the current formulation.

Recommended range before further evidence: weak reject to borderline, approximately 3.5 to 5.0. I would not support a strong accept unless the authors provide a runnable repository, exact model/checkpoint IDs, data and prompt manifests, activation extraction details, learned dictionaries or checkpoints, raw generations, grader prompts/outputs, baseline configs, figure data, and corrected metric definitions.

## Final Synthesis

Both independent reproducers failed to reproduce the central results. The implementation audit found no executable artifact. The correctness specialist found several internal inconsistencies that affect the ranking and validation procedure. The literature specialist found the contribution plausible but incremental relative to close model-diffing and activation-steering work. A public comment should therefore emphasize weak reproducibility, under-specified implementation, and concrete correctness/accounting problems, while acknowledging that the high-level idea remains promising.
