# SurrogateSHAP role findings

## Reproducibility lead
- Central claim: a training-free proxy game plus TreeSHAP yields faithful, efficient contributor attribution for T2I diffusion models.
- Reproduction target: the proxy-fidelity proposition and its use in justifying the error decomposition behind LDS/counterfactual results.
- Bottom line: the released artifacts are manuscript-only, and the most decision-relevant issue I could verify directly is a notation/logic break in the proxy-fidelity theory rather than an executable rerun.

## Reproducer A
- Unpacked the Koala tarball at `tmp/cb932990/src`. It contains TeX, figures, and tables, but no runnable implementation.
- Opened [method.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/method.tex:5) and verified that the method defines coalition-specific mixtures:
  `p_{theta*_S}(x) = sum_{y in Y_S} pi_S(y) p_{theta*_S}(x|psi(y))`
  and
  `hat v_theta(S) = F(sum_{y in Y_S} pi_S(y) p_theta(x|psi(y)))`.
- This makes the coalition prior `pi_S` and the coalition label set `Y_S` load-bearing objects.

## Reproducer B
- Followed the stated proof path in [method.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/method.tex:42) and [appendix.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/appendix.tex:182).
- The proposition restates the objects as `v(S):=F(p_{theta*_S})` and `hat v_theta(S):=F(p_theta)` at [method.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/method.tex:43).
- The proof then defines `nu_S` and `hat nu_S` using a fixed global conditioning prior `pi` and the unconditional target conditional `p_theta(.|c)` rather than a coalition-restricted prior `pi_S` or mixture distribution `P_S`; see [appendix.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/appendix.tex:183) and [appendix.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/appendix.tex:187).
- I could not find a later definition that reconciles `pi` with `pi_S` or explains why the coalition dependence can be dropped without changing the game.

## Implementation auditor
- `tar -tzf tmp/cb932990/paper.tar.gz` lists only manuscript assets plus figures/tables; no `.py`, `.sh`, notebook, or config artifacts.
- The appendix does provide some training details, including LoRA rank 256 for ArtBench, rank 128 for FLUX, and an XGBoost grid over depth/trees/lr/L1 at [appendix.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/appendix.tex:292) and [appendix.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/appendix.tex:330).
- However, there is still no code path that lets a reviewer verify whether the theoretical object implemented in experiments is the coalition-restricted mixture from Eq. 2 or the simplified object used in the proof.

## Correctness specialist
- The proxy-fidelity proposition is supposed to control `E_gap` in the Shapley error decomposition; see [method.tex](/home/mila/l/lia/peer-review-agents/agent_configs/agent1/tmp/cb932990/src/sections/method.tex:129).
- If the theorem only bounds a global `F(p_theta)` object while experiments use coalition-specific `hat v_theta(S)`, then the main bridge from the practical proxy to the retraining game is underspecified.
- This matters most for ArtBench/Fashion, where contributors correspond to artist/brand prompt subsets and the coalition prior is not a cosmetic detail.

## Literature specialist
- I did not use post-publication signals or conference outcomes.
- Within the paper’s own framing, the main novelty is the training-free coalition proxy plus tree-surrogate Shapley pipeline. That makes the coalition-specific theoretical justification part of the core contribution, not a peripheral appendix issue.

## Score impact
- This is a moderate-to-serious correctness/reproducibility concern. It does not prove the empirical results are wrong, but it weakens the paper’s central claim that the proposed proxy is theoretically justified as an approximation to the retraining game.
- What would change my view: an explicit correction showing the proposition/proof with coalition-restricted `pi_S` or `P_S`, plus clarification that the experiments actually use that same object.
