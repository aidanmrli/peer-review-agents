# Correctness Specialist Report

Paper: `75c4a4bd-208f-451a-8ed8-121748a738c7`

Title: "Plain Transformers are Surprisingly Powerful Link Predictors"

Role: Correctness Specialist

Sources inspected:

- `artifacts/main.tex`
- Appendix embedded in `artifacts/main.tex`
- Tables and figure captions in `artifacts/main.tex`
- Local role instructions in `skills/correctness-specialist.md`

Commands / checks used:

- `rg` and `nl -ba ... | sed -n ...` to inspect theorem statements, proofs, architecture equations, table entries, and appendix sections.
- Manual and small arithmetic recomputation of table deltas from `main.tex`, including the multiplicative-residual gain row and best-model comparisons.
- Image files were enumerated for scope, but no quantitative figure data were available in source form.

## Bottom Line

The empirical tables are not internally fabricated on their face, and the multiplicative-residual gain row is arithmetically correct. However, the paper has serious correctness defects in its theoretical chain and in one key evaluation protocol claim. The most important technical error is algebraic: the proofs of the NBFNet degeneration and local-heuristic estimation set the Transformer block output to zero, which makes the entire residual propagation branch zero rather than a message-passing update. This invalidates the stated proof route for PENCIL realizing NBFNet-style propagation, global heuristics, and local heuristic estimators. Separately, the claimed HeaRT result on `ogbl-ppa` is not actually evaluated under the stated curated-negative HeaRT protocol according to the appendix.

## Fatal / Major Issues

### 1. The NBFNet degeneration proof collapses PENCIL to zero, not to message passing

Candidate error: The proof of Proposition 1 says that setting the Transformer block to output zero removes attention and makes Eq. (2) reduce to propagation on the reconstructed adjacency.

Exact location:

- Architecture update: `artifacts/main.tex:214-220`
- Proposition statement: `artifacts/main.tex:270-273`
- Proof: `artifacts/main.tex:615-621`

Why it is technically wrong:

The model update is

```text
Z^(k) = T_k(H^(k-1))
H^(k) = Z^(k) + P_k(A~ Z^(k)).
```

If `T_k` maps every token to zero, then `Z^(k)=0`, `A~ Z^(k)=0`, and therefore `H^(k)=0 + P_k(0)`. Under the usual linear interpretation of `P_k`, this is zero or at most a learned bias if biases exist; it is not propagation of `H^(k-1)` over `A~`. Thus the proof does not establish reduction to a source-conditioned MPNN.

Evidence / derivation:

- From `artifacts/main.tex:216-218`, propagation is applied to `Z^(k)`, not directly to `H^(k-1)`.
- From `artifacts/main.tex:619-620`, the authors set `T_k` to zero and then claim Eq. (2) reduces to adjacency propagation. Algebraically, the adjacency branch receives the zero tensor.

Severity: Fatal for Proposition 1 and any downstream theorem/corollary relying on it; major for the paper overall.

Consequence for acceptance:

The theoretical bridge from PENCIL to NBFNet-style propagation is not proven. This materially weakens the paper's central claim that the architecture has a formal mechanism for realizing path-based link prediction algorithms.

### 2. The local-heuristic estimation proof has the same zero-propagation error

Candidate error: The proof of Proposition 2 repeats the same move: it sets `T_k` to zero and claims Eq. (2) becomes a sum-aggregation MPNN satisfying the common-neighbor estimator assumptions.

Exact location:

- Proposition statement: `artifacts/main.tex:288-290`
- Restated common-neighbor theorem: `artifacts/main.tex:639-655`
- Proof: `artifacts/main.tex:657-665`

Why it is technically wrong:

The common-neighbor estimator requires nonzero node signatures to be aggregated by sum message passing. But after setting `T_k=0`, Eq. (2) has no nonzero node signatures in the residual branch. It therefore cannot produce the single-layer or multi-layer sum-aggregation embeddings required by the cited theorems.

Evidence / derivation:

- `artifacts/main.tex:664` states that setting `T_k` to zero reduces Eq. (2) to a sum-aggregation MPNN.
- Direct substitution into `artifacts/main.tex:216-218` gives `H^(k)=P_k(A~0)` rather than `sum_neighbors H^(k-1)`.

Severity: Fatal for Proposition 2; major for the paper overall.

Consequence for acceptance:

The claimed formal support for estimating local overlap heuristics is not established. This directly undermines the "theoretical unification" contribution at `artifacts/main.tex:155` and the abstract's claim that PENCIL implicitly generalizes a broad class of heuristics (`artifacts/main.tex:132`).

### 3. The corollary claiming exact realization of global graph algorithms is not justified under the stated PENCIL setting

Candidate error: The paper claims PENCIL can realize Katz, Personalized PageRank, shortest-path distance, widest path, and most reliable path.

Exact location:

- Corollary statement: `artifacts/main.tex:275-277`
- Proof: `artifacts/main.tex:623-633`
- Local-subgraph restriction: `artifacts/main.tex:151`, `artifacts/main.tex:227`, `artifacts/main.tex:440`

Why it is technically wrong or unsupported:

Even if the NBFNet degeneration proof were fixed, the corollary imports a full-graph generalized Bellman-Ford result without showing that PENCIL's actual computation has access to the full graph, enough iterations, appropriate semiring operations, and edge weights. The paper repeatedly defines PENCIL as operating on fixed-budget sampled local subgraphs. Exact Katz, PPR, and SPD are full-graph quantities in general; two candidate pairs can induce the same sampled local subgraph while differing in paths or PageRank mass outside the sample.

Evidence / derivation:

- `artifacts/main.tex:227` explicitly states that the heuristic target is computed on the full graph while models are restricted to an induced subgraph centered around `(u,v)`.
- `artifacts/main.tex:440` states GPU compute and memory scale with context subgraphs per candidate link, again confirming local-subgraph evaluation.
- `artifacts/main.tex:631-632` cites NBFNet/Bellman-Ford recovery, but does not prove that Eq. (2) implements the required semiring recurrences or has the required graph support.

Severity: Major to fatal for the global-heuristics theoretical claim.

Consequence for acceptance:

The paper can still claim empirical approximation of full-graph heuristic labels, but it cannot claim exact realization of these global algorithms under the stated local sampled-subgraph PENCIL model.

### 4. The `ogbl-ppa` HeaRT result is not evaluated under the stated HeaRT protocol

Candidate error: The main text presents Table 2 as HeaRT evaluation with curated negative links per positive test example, but the appendix says `ogbl-ppa` uses a single negative link per positive and reuses the checkpoint from the original benchmark setting.

Exact location:

- HeaRT protocol description: `artifacts/main.tex:414`
- HeaRT top-score claim: `artifacts/main.tex:416`
- Appendix exception for `ogbl-ppa`: `artifacts/main.tex:849`
- HeaRT table entries: `artifacts/main.tex:374-404` and complete table `artifacts/main.tex:935-965`

Why it is technically wrong or unsupported:

The main text says HeaRT differs by curated negatives per positive test example. For `ogbl-ppa`, the appendix states that the authors instead use one negative link for each positive link because the HeaRT protocol is computationally prohibitive, and they use the optimal checkpoint from the original benchmark setting. This is not the same evaluation protocol described in the main text and makes the `ogbl-ppa` HeaRT comparison non-uniform.

Evidence / derivation:

- `artifacts/main.tex:414` defines the HeaRT setting as curated negatives per positive test example.
- `artifacts/main.tex:849` explicitly exempts `ogbl-ppa` from that setup.
- The paper's main HeaRT performance claim includes `ogbl-ppa` as a top-score result at `artifacts/main.tex:416`.

Severity: Major.

Consequence for acceptance:

The claimed HeaRT state-of-the-art result on `ogbl-ppa` should not be treated as a clean HeaRT result unless the authors report the actual HeaRT negative set evaluation or clearly separate this nonstandard single-negative protocol from the HeaRT table.

### 5. The "plain Transformer" characterization is contradicted by the architecture and ablation

Candidate error: The title, abstract, and conclusion characterize PENCIL as a plain/vanilla Transformer, but the model includes an explicit graph propagation residual at every layer.

Exact location:

- Abstract plain-Transformer claim: `artifacts/main.tex:132`
- Introduction claim: `artifacts/main.tex:151`, `artifacts/main.tex:153`
- Multiplicative residual equation: `artifacts/main.tex:214-220`
- Appendix adjacency operator and row-normalized propagation: `artifacts/main.tex:477-496`
- Ablation conclusion that explicit structural prior is necessary: `artifacts/main.tex:984`
- Conclusion "vanilla Transformer" claim: `artifacts/main.tex:438`

Why it is technically wrong or unsupported:

Eq. (2) adds `P_k(A~ Z^(k))`, an explicit adjacency-matrix propagation branch. The appendix further states that the implemented operator adds identity links, gives task tokens receive-only behavior, and empirically uses row-normalized `D^{-1}A~`. The ablation then concludes that this explicit structural prior is necessary for every layer. This is a graph-specific propagation module, not merely a plain encoder-only Transformer over tokens.

Evidence / derivation:

- `artifacts/main.tex:220` calls the second branch an explicit matrix-multiplication residual.
- `artifacts/main.tex:496` describes explicit one-hop structure-respecting aggregation.
- `artifacts/main.tex:984` says input encodings alone are suboptimal and an explicit structural prior is necessary for every layer.

Severity: Major.

Consequence for acceptance:

The empirical method may be useful, but the paper's headline framing overstates the simplicity/plainness of the architecture. The contribution should be described as a Transformer plus explicit adjacency-propagation residual over sampled subgraphs.

## Moderate Issues

### 6. Distributional permutation invariance is narrower than the text claims

Candidate error: The paper says the randomized predictor remains a valid graph function because relabeling leaves the predictor unchanged in distribution.

Exact location:

- Non-deterministic invariance discussion: `artifacts/main.tex:247-260`
- Formal proof: `artifacts/main.tex:507-586`
- Actual encoding with task tokens and role flags: `artifacts/main.tex:191-200`, `artifacts/main.tex:463-496`

Why it is technically wrong or unsupported:

The theorem proves only equality in distribution for an abstract predictor `f(P_rho A P_rho^T)`. It does not show deterministic invariance of the actual inference procedure for a single sampled ordering, nor does it cover all implementation details: task-token duplication, role flags, optional node features, padding, subgraph sampling, and the adjacency residual operator. Equality in distribution is useful, but it is weaker than a deterministic graph function unless the inference rule averages over the randomization or otherwise fixes a canonical invariant estimator.

Evidence / derivation:

- `artifacts/main.tex:247` admits PENCIL is not deterministically invariant.
- `artifacts/main.tex:250-257` and `artifacts/main.tex:507-527` prove distributional equality only.
- `artifacts/main.tex:260` then calls the randomized predictor a valid graph function, which overstates what has been proven for a single model evaluation.

Severity: Moderate.

Consequence for acceptance:

The invariance analysis is not wrong as a distributional theorem, but the conclusion should be narrowed. It does not guarantee stable deterministic link scores under relabeling for the implemented model.

### 7. Main performance conclusions overgeneralize from the reported tables

Candidate error: The abstract and results section imply broad outperformance over heuristic-informed GNNs and consistently lower variance.

Exact location:

- Abstract broad outperformance claim: `artifacts/main.tex:132`
- Performance discussion: `artifacts/main.tex:416`
- Main original-setting table: `artifacts/main.tex:331-368`
- Main HeaRT table: `artifacts/main.tex:374-404`
- Complete original table: `artifacts/main.tex:896-933`
- Complete HeaRT table: `artifacts/main.tex:935-965`

Why it is technically wrong or unsupported:

Using the best PENCIL variant from the table and the best non-PENCIL baseline in each dataset:

- Original setting: PENCIL is best on `cora` by +2.81 and `ogbl-ppa` by +1.13, but trails the best non-PENCIL result on `citeseer` by -17.91, `pubmed` by -6.39, `ogbl-collab` by -1.26, and `ogbl-citation2` by -3.86.
- HeaRT: PENCIL is best on `ogbl-ppa` by +4.03 and `ogbl-ddi` by +0.61, but trails on `cora` by -2.17, `citeseer` by -11.85, `pubmed` by -1.11, `ogbl-collab` by -2.22, and `ogbl-citation2` by -1.27.

The variance claim is also too broad. For example, in the original table, PENCIL on `pubmed` has standard deviations 2.59 or 5.14, while LPFormer is 1.92 and NBFNet is 2.12. On `ogbl-citation2`, PENCIL has 0.20 or 0.26, while NCN has 0.05 and Refined-GAE has 0.06.

Severity: Moderate.

Consequence for acceptance:

The tables support a narrower claim: PENCIL is very strong on a subset of datasets, especially `ogbl-ppa`, but it does not broadly outperform heuristic-informed or ID-based methods across the benchmark suite.

### 8. The node-feature conclusion is too broad for OGB datasets

Candidate error: The paper claims features provide necessary signal on OGB datasets, yielding major gains on `ogbl-ppa`.

Exact location:

- Structural-sufficiency discussion: `artifacts/main.tex:418`
- Original table: `artifacts/main.tex:350-365`
- HeaRT table: `artifacts/main.tex:387-401`

Why it is technically wrong or unsupported:

The reported table values show material feature benefit mainly on `ogbl-ppa`, not generally across OGB datasets.

Recomputed feature deltas from the table:

- Original `cora`: PENCIL with features 32.12 vs without features 42.23, delta -10.11.
- Original `ogbl-ppa`: 79.54 vs 73.85, delta +5.69.
- Original `ogbl-collab`: 66.56 vs 66.88, delta -0.32.
- Original `ogbl-citation2`: 86.86 vs 86.74, delta +0.12.
- HeaRT `ogbl-ppa`: 45.43 vs 44.57, delta +0.86.
- HeaRT `ogbl-collab`: 5.40 vs 5.25, delta +0.15.
- HeaRT `ogbl-citation2`: 23.43 vs 23.36, delta +0.07.

Severity: Moderate.

Consequence for acceptance:

The correct conclusion is that features materially help on `ogbl-ppa` in the original setting and have small or mixed effects elsewhere. The broader statement that features provide necessary OGB signal is not supported.

### 9. The adjacency operator used in the residual is underspecified and inconsistent between main text and appendix

Candidate error: The main text describes reconstructing raw adjacency from the input, while the appendix defines an operator that adds identity links, appends task-token zero columns, and uses row normalization empirically.

Exact location:

- Main adjacency description: `artifacts/main.tex:207-210`
- Residual equation: `artifacts/main.tex:214-220`
- Appendix reconstruction: `artifacts/main.tex:466-496`

Why it is technically wrong or unsupported:

The main text says normalized variants are applicable but uses raw `A~` for notational simplicity. The appendix, however, defines `A~_src = X_adj + X_id`, appends zero task columns, and states that experiments use row-normalized `D^{-1}A~`. This changes the actual operator from a raw subgraph adjacency to a self-looped, task-receiving, row-normalized context-sourced operator.

Evidence / derivation:

- `artifacts/main.tex:475-482` adds the identifier slice to adjacency.
- `artifacts/main.tex:488-496` appends two all-zero task-token columns.
- `artifacts/main.tex:496` states row normalization is used empirically.

Severity: Moderate.

Consequence for acceptance:

The mathematical claims and reproducibility of Eq. (2) depend on the actual operator. The paper should state the implemented operator in the main method and align the proofs with that operator.

### 10. The local-heuristic proof's orthogonality premise conflicts with the claimed sampled-token advantage

Candidate error: The main text argues that PENCIL needs only `N_max` well-separated vectors, but the proof of Proposition 2 states orthonormal vectors can be chosen for `d >= |V|`.

Exact location:

- Sampled-token advantage claim: `artifacts/main.tex:292-316`
- Local-heuristic proof: `artifacts/main.tex:663-664`

Why it is technically wrong or unsupported:

The proof invokes a global node-vector condition (`d >= |V|`) even though the claimed advantage is that PENCIL only needs slot/token vectors for a sampled subgraph (`N_max << |V|`). These are different constructions. If vectors are per global node, the claimed memory/dimension advantage disappears. If vectors are per sampled slot, the authors need a separate argument that the cited unbiased estimator applies when global nodes are randomly assigned fresh slot vectors per candidate subgraph.

Severity: Moderate.

Consequence for acceptance:

The proof does not cleanly establish the advertised reason PENCIL inherits MPLP-style unbiased heuristic estimation while avoiding global node signatures.

## Minor Issues / Correct Checks

### A. Multiplicative-residual gain row is arithmetically correct

Exact location:

- Table: `artifacts/main.tex:969-981`

Recomputed deltas:

- `cora` MRR: 42.23 - 34.64 = +7.59.
- `cora` H@3: 43.34 - 35.98 = +7.36.
- `cora` H@20: 67.32 - 58.67 = +8.65.
- `pubmed` MRR: 38.28 - 21.49 = +16.79.
- `pubmed` H@3: 43.61 - 21.81 = +21.80.
- `pubmed` H@20: 57.49 - 31.81 = +25.68.
- `ogbl-collab` H@20: 53.47 - 49.52 = +3.95.
- `ogbl-collab` H@50: 66.88 - 53.43 = +13.45.
- `ogbl-collab` H@100: 69.75 - 56.04 = +13.71.

Severity: No error.

Consequence for acceptance:

This table supports the narrower empirical claim that the residual branch helps on the listed datasets and metrics.

### B. Welch-bound monotonicity proof is correct

Exact location:

- Proposition: `artifacts/main.tex:312-314`
- Proof: `artifacts/main.tex:667-689`

Evidence:

For `M > N > d >= 2`, the proof compares `(M-d)/(M-1)` and `(N-d)/(N-1)` and obtains `(M-N)(d-1)>0`, which is correct. Since the square root and factor `1/d` preserve monotonicity, `W(N,d)` is strictly increasing in `N` for `N>d`.

Severity: No error.

Consequence for acceptance:

This specific mathematical lemma is sound, though its use in the local-heuristic proof is weakened by Issue 10.

### C. There are no algorithm environments to audit beyond the generalized Bellman-Ford definition

Exact location:

- Only algorithm-like formalism found: `artifacts/main.tex:592-607`

Assessment:

The generalized Bellman-Ford definition itself is standard at a high level. The problem is not the definition, but the unsupported transfer from this semiring recurrence to PENCIL's Eq. (2) update.

Severity: No standalone error.

## Final Synthesis and Score Impact

The paper contains useful empirical evidence that an adjacency-row Transformer with an explicit propagation residual can be competitive, especially on `ogbl-ppa`, and the residual ablation is internally consistent. The acceptance case is materially weakened by the proof failures and protocol mismatch. The theoretical claims connecting PENCIL to NBFNet, local heuristic estimators, and global graph algorithms are not correct as written. The `ogbl-ppa` HeaRT claim should be separated from true HeaRT evaluation because the appendix says it uses a different negative-sampling protocol. I would assign a substantial correctness downgrade unless the authors revise the theory to use a nonzero propagation input, narrow the global-heuristic claims to approximation under sampled subgraphs, and clearly relabel the nonstandard `ogbl-ppa` HeaRT experiment.
