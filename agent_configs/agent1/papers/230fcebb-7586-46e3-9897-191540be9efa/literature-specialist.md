# Literature Specialist Report

Paper: 230fcebb-7586-46e3-9897-191540be9efa  
Title: Why Depth Matters in Parallelizable Sequence Models: A Lie Algebraic View  
Role: Literature Specialist, agent1  
Date: 2026-04-24

## Scope and Permitted Sources

I used only the submitted paper artifacts, the linked official repository, and primary prior work available before or at the paper release. I did not use OpenReview reviews, decisions, citation counts, later discussion, social media, or post-publication commentary about this submission.

Paper/artifact sources inspected:

- Koala paper metadata: `get_paper(230fcebb-7586-46e3-9897-191540be9efa)`, which returned arXiv ID 2603.05573, PDF/source tarball, and repository `https://github.com/kazuki-irie/lie-algebra-state-tracking`.
- Local source artifacts: `papers/230fcebb-7586-46e3-9897-191540be9efa/artifacts/main.tex`, `A1_Preliminaries.tex`, `A2_Proofs.tex`, `A3_Experiments.tex`, `main.bib`, `main_2.bib`.
- Linked repo clone in `/tmp/koala-lit-230fcebb`, HEAD `e6575fae9d3aa8f32cf30254269054fbf58c87b1`.

Primary prior work checked:

- Liu et al., *Transformers Learn Shortcuts to Automata*, arXiv:2210.10749, https://arxiv.org/abs/2210.10749.
- Merrill and Sabharwal, *The Parallelism Tradeoff: Limitations of Log-Precision Transformers*, arXiv:2207.00729, https://arxiv.org/abs/2207.00729.
- Merrill, Petty, and Sabharwal, *The Illusion of State in State-Space Models*, arXiv:2404.08819, https://arxiv.org/abs/2404.08819.
- Hu, Liu, and Jin, *On Limitation of Transformer for Learning HMMs*, arXiv:2406.04089, https://arxiv.org/abs/2406.04089.
- Cirone et al., *Theoretical Foundations of Deep Selective State-Space Models*, arXiv:2402.19047, https://arxiv.org/abs/2402.19047.
- Walker et al., *Log Neural Controlled Differential Equations: The Lie Brackets Make a Difference*, arXiv:2402.18512, https://arxiv.org/abs/2402.18512.
- Walker et al., *Structured Linear CDEs: Maximally Expressive and Parallel-in-Time Sequence Models*, arXiv:2505.17761, https://arxiv.org/abs/2505.17761.
- Shakerinava et al., *The Expressive Limits of Diagonal SSMs for State-Tracking*, arXiv:2603.01959, https://arxiv.org/abs/2603.01959.
- Grazzi et al., *Unlocking State-Tracking in Linear RNNs Through Negative Eigenvalues*, arXiv:2411.12537, https://arxiv.org/abs/2411.12537.
- Siems et al., *DeltaProduct: Increasing the Expressivity of DeltaNet Through Products of Householders*, arXiv:2502.10297, https://arxiv.org/abs/2502.10297.
- Karuvally et al., *Bridging Expressivity and Scalability with Adaptive Unitary SSMs*, arXiv:2507.05238, https://arxiv.org/abs/2507.05238.
- Classical literature cited by the paper on Lie/Magnus/control/Krohn-Rhodes: Jurdjevic and Sussmann 1972; Krener 1977; Iserles et al. 2000/2008; Blanes et al. 2009; Krohn and Rhodes 1965/1968 as mediated by the paper's Krohn-Rhodes discussion and Maler 2010.

## Novelty Claim Checked

The paper's novelty claim is that it gives a Lie-algebraic control view of why depth matters in parallelizable sequence models, connecting model depth to towers of Lie algebra extensions; it characterizes constant-depth/restricted sequence-model expressivity; and it derives a depth-dependent approximation error bound, with local error scaling as `O(epsilon^(2^(k-1)+1))` for non-solvable systems. In the source this appears in the abstract and introduction (`main.tex:96-121`), in the theory overview (`main.tex:422-427`), in the single-layer error bound (`main.tex:438-445`), in the depth/derived-length theorem (`main.tex:458-476`), in the non-solvable approximation corollary (`main.tex:506-516`), and in the bounded word-problem logarithmic-depth proposition (`main.tex:520-529`).

Bottom line: the Lie/Magnus error-scaling formulation is the credible novel center. The broader message that depth matters for parallelizable state tracking, that logarithmic depth suffices for bounded automata/word problems, that solvable structure admits depth-efficient simulation, and that diagonal/commuting SSMs have state-tracking limits is already strongly represented in prior work. The paper is better framed as a synthesis and quantitative continuous-control error analysis, not as the first explanation of depth-dependent state-tracking expressivity.

## Prior Work Considered and Overlap

Liu et al. already prove the central bounded-length Transformer/semiautomaton shortcut facts. Their Theorem 1 gives a depth `ceil(log2 T)` Transformer that simulates any semiautomaton at length `T`, and their Krohn-Rhodes result gives constant-depth shortcuts for solvable semiautomata while ruling out general non-solvable constant-depth shortcuts unless `TC^0 = NC^1`. They also document brittleness of shortcuts under distribution shift and length generalization. The present paper acknowledges this in the related work and in the footnote under Proposition `col:logdepth`, but its Proposition `col:logdepth` is not novel as a high-level depth result; its distinction is the Lie-free-algebra/Lyndon-word construction and SSM formulation (`main.tex:520-529`, `A2_Proofs.tex:492-539`).

Merrill et al. already extend the circuit-complexity limitation to SSMs. They show common SSM variants, including S4/S6/Mamba-style diagonal structures, are in uniform `TC^0` and therefore cannot solve `NC^1`-hard state-tracking problems such as permutation/word problems under the usual separation assumption. They also use A5/S5-style word-problem benchmarks and plot minimum depth for `>90%` validation accuracy as sequence length grows. The current paper's A5 depth experiment and word-problem tagging formulation are therefore not new benchmark territory; they reuse and broaden a known testbed (`main.tex:608-615`, `main.tex:655-669`). The current contribution is to attach a Lie/Magnus approximation interpretation to the observed depth trend.

Hu et al. already connect Transformer depth to maximum learnable sequence length for HMM-like systems and prove logarithmic-depth approximation results for HMMs. The current paper cites Hu correctly (`main.tex:525-526`, `main.tex:742-745`), but its MDP/world-model motivation should be framed with care: the submitted experiments are fully observed group-element state-tracking and A5 rotation regression, not hidden-state belief filtering. Hu's distinction matters because partial observability and uninformative observations introduce learnability failures beyond the algebraic order-sensitivity story.

Shakerinava et al. are extremely close on diagonal SSM state tracking. Their paper proves that a single-layer input-dependent complex diagonal SSM tracks exactly Abelian groups at finite precision, and that a `k`-layer DCD SSM tracks a group iff the group has a subnormal series of length `k` with Abelian factors. They also report the expressivity-learnability gap for non-Abelian solvable groups. The submitted paper cites this work only as `anonymous2025the` in the source (`main.tex:430`, `main.tex:761`), but the public arXiv version exists before this review cycle and should be de-anonymized if allowed by the submission policy. This prior work materially narrows the novelty of the paper's solvable-depth/group-tracking claims. The submitted paper is distinct only where it moves from finite group state-tracking in diagonal complex SSMs to finite-dimensional Lie algebras, continuous controlled flows, and non-solvable approximation via Magnus truncation.

The rough-path/CDE line already uses signatures, log-signatures, Lie brackets, and structured linear controlled differential equations to analyze SSM expressivity. Cirone et al. characterize expressive power of selective SSMs and show diagonal recurrences are weaker than dense ones, with stacking recovering higher-order statistics. Walker et al. 2025 propose SLiCEs, prove expressivity for structured linear CDEs, and test A5 state tracking and formal-language length generalization. The submitted paper cites Walker et al. 2025 but does not experimentally compare against SLiCE-style structured non-diagonal parallel-in-time baselines. This is a notable omission because SLiCE is directly positioned as a parallelizable structured-state alternative that can solve A5 with a single layer in some variants, whereas the current experiments mainly contrast Transformer/GLA/Signed Mamba/AUSSM against DeltaProduct.

Classical Lie, Magnus, and control theory already supply the core mathematical devices. The use of commutators as order-sensitivity, Magnus terms as approximation/truncation error, and cascade/extension decompositions is not new mathematics. The paper is generally honest about this, citing Iserles, Blanes, Jurdjevic, Krener, and related texts. The strongest novelty is in applying these tools to the specific architecture/expression question for parallelizable sequence models and deriving the specific local depth-error order in the SSM setting.

Krohn-Rhodes is relevant enough that the bibliography should be stronger. The source bibliography contains Maler 2010 on the Krohn-Rhodes cascaded decomposition theorem but does not appear to cite Krohn and Rhodes 1965 or the later finite semigroup complexity sources directly (`rg "Krohn|Rhodes"` found only `maler2010krohn` in `main.bib`). Since the paper's depth-as-cascade/extension intuition is explicitly adjacent to algebraic automata theory, the original Krohn-Rhodes sources or a standard algebraic automata text should be included, not only mediated through Liu et al. and Maler.

## Overlap and Distinction by Claim

Claim: "Depth matters for parallelizable sequence models."  
Status: already established in multiple forms. Liu et al. show log-depth shortcuts and constant-depth solvable cases for Transformers; Merrill et al. show SSM/Transformer depth must grow for hard state tracking; Hu et al. empirically and theoretically connect Transformer depth to learnable HMM sequence length. The submitted paper's contribution is not the existence of the depth phenomenon.

Claim: "Constant-depth/restricted models correspond to limited algebraic classes."  
Status: heavily anticipated. Circuit-complexity results place Transformers and common SSMs in `TC^0`; Shakerinava et al. give an exact finite-precision group-theoretic characterization for `k`-layer complex diagonal SSMs. The submitted paper's Lie-algebra version is broader in continuous-control language but narrower in finite-precision operational consequences.

Claim: "Abelian `k`-layer SSMs simulate systems whose Lie algebra has derived length `k`."  
Status: plausible distinct formulation, but it echoes known cascade decomposition and the diagonal-SSM solvable-group characterization. The paper itself notes that Krener proved split extensions admit cascade decomposition (`A2_Proofs.tex:433-434`). The novelty should be presented as a bridge between solvable Lie algebra extension theory and SSM depth, not as an entirely new algebraic depth principle.

Claim: "Non-solvable local approximation error decays as `O(epsilon^(2^(k-1)+1))`."  
Status: this is the most distinctive literature contribution. It combines Magnus truncation/nilpotentization with the derived-length/depth construction to give a quantitative local error order (`main.tex:506-516`, `A2_Proofs.tex:436-452`). Prior CDE/rough-path work explains why Lie brackets matter and gives log-signature machinery, but I did not find a prior sequence-model paper among the checked sources that states this exact depth-to-Magnus-order error law for parallelizable SSM approximations.

Claim: "Experiments validate the theory."  
Status: partly supported but not novel as benchmark framing. The word-problem and A5 state-tracking benchmark lineage is Merrill/Liu/DeltaProduct/Grazzi/AUSSM/SLiCE. The current paper adds a continuous A5 rotation regression variant and a coherent Lie-theoretic story, but the experimental comparison is incomplete for literature positioning because RNN/IDS4/SLiCE-style baselines are absent, and the repo instructions confirm only Transformer, GLA, Mamba variants, AUSSM, and DeltaProduct as primary models (`state_tracking/readme.md:66`, repo README lines 9-13).

## Missing Citations or Baselines

1. De-anonymize and foreground Shakerinava et al. 2026 if policy allows. Its exact `k`-layer diagonal SSM solvable-group characterization is too close to leave as a minor anonymous citation.

2. Add primary Krohn-Rhodes sources or a standard algebraic automata reference. The current source only shows Maler 2010 for Krohn-Rhodes; the original Krohn and Rhodes 1965/1968 finite semigroup/machine decomposition results should be cited if Krohn-Rhodes is used as a conceptual pillar.

3. Compare against SLiCE or explicitly justify omission. Walker et al. 2025 are cited, but their SLiCE models are directly about structured parallel-in-time CDE/SSM expressivity and include A5 and formal-language state-tracking experiments. A baseline or a clear "out of scope because ..." statement is needed.

4. Include a non-parallel recurrent reference baseline or explain why it is excluded. Merrill et al. use RNN/IDS4 as expressive counterpoints. Since the paper claims guidance for model choice, the absence of RNN/IDS4/structured non-diagonal comparisons weakens the empirical novelty and practical framing.

5. Clarify finite precision versus real arithmetic against the diagonal-SSM literature. The paper mentions finite precision as future work (`main.tex:735`) while Shakerinava et al. and Grazzi et al. make finite precision central. Because the submitted theorems rely on real analytic/smooth maps and real arithmetic, the relation to finite-precision state-tracking limits must be clearer.

6. Be precise about "permutation invariance/order symmetry" in Transformers. Self-attention without positional information is permutation equivariant/invariant in a relevant sense, but practical causal Transformers include masks and positional encodings. The paper later acknowledges multiplicative positional encoding (`main.tex:729-731`), but the introduction's broad statement (`main.tex:104-109`) should be sharpened to avoid overgeneralization.

## Framing Accuracy

The paper's related-work section is mostly accurate in naming the right literatures. It correctly credits Liu for semiautomata/Krohn-Rhodes/log-depth and brittleness, Hu for HMM depth-length observations, Merrill for SSM circuit-complexity limits, DeltaProduct/Grazzi/AUSSM for state-tracking architecture variants, and CDE/rough-path work for the Lie/signature perspective.

The framing is nevertheless too expansive in two places. First, "the error-expressivity scaling laws have not ... been clearly derived" is defensible only if restricted to the paper's specific Magnus/nilpotentization local error bound. It is not defensible as a broad statement about depth-dependent state-tracking scaling, because Liu, Merrill, Hu, and Shakerinava et al. already provide depth/length or depth/solvable-class scaling results. Second, "deep architectures compensate to rapidly minimize the error" should be presented as a local analytic result plus partial empirical trend, not as a general empirical conclusion. The paper's own experiments show optimization failures, depth saturation, and learnability gaps (`main.tex:635-643`, `main.tex:667-713`), consistent with the prior diagonal-SSM literature.

The continuous Lie-algebraic view is valuable, but it should be framed as a unifying lens and quantitative approximation theorem layered on top of known automata, circuit, and CDE results. It is not a cleanly standalone novelty story.

## Decision Impact

Literature impact is mixed. I would give positive credit for the Lie/Magnus derivation of a local depth-dependent approximation error order and for connecting solvable Lie algebra extensions to SSM depth in one coherent theoretical narrative. That is a real contribution if the correctness specialist verifies the assumptions and proof details.

The novelty case is materially weaker than the abstract and introduction imply. The central "depth matters" message, logarithmic-depth bounded word-problem result, finite-depth solvable-state-tracking story, and state-tracking benchmarks are all substantially anticipated. The paper should be judged on whether its specific Lie/Magnus error bound is correct, useful, and empirically meaningfully supported. On literature grounds alone, I would mark the paper down from a strong-accept framing to a borderline/weak-accept framing unless the theory is exceptionally clean and the final version explicitly narrows its novelty claims and improves baseline/citation coverage.

Score impact from literature review: material downgrade for overstated novelty and incomplete baseline positioning; not a literature-only rejection because the quantitative Lie/Magnus error law appears distinct among the checked primary sources.

## Evidence and Commands

Commands used included:

- `sed -n '1,220p' skills/literature-specialist.md`
- `curl -fsSL https://koala.science/skill.md | sed -n '1,240p'`
- Koala MCP `get_paper` for the paper metadata and linked artifacts.
- `rg -n "(Krohn|Merrill|Liu|Hu|semiautomata|constant-depth|circuit|Magnus|Lie|state-tracking|word problem)" ...` over the submitted LaTeX sources and bibliographies.
- `nl -ba .../main.tex | sed -n '96,132p'`, `sed -n '422,540p'`, and `sed -n '588,762p'` for exact paper locations.
- `git clone --depth 1 https://github.com/kazuki-irie/lie-algebra-state-tracking /tmp/koala-lit-230fcebb` and repo README/source inspection.
- `curl -fsSL https://arxiv.org/e-print/<id>` followed by extraction and `rg` over primary prior-work LaTeX sources for Liu, Merrill, Hu, Cirone/Muca, Walker, Shakerinava, Grazzi, DeltaProduct, and AUSSM.

Reproduction note: this role did not run empirical reproduction. It inspected the linked repository for artifact and baseline scope only. Independent empirical reproduction outcomes should be supplied by the Independent Reproducer and Implementation Auditor role reports.
