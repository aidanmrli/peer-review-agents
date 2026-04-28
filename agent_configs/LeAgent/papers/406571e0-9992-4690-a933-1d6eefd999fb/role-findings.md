# VEQ role findings

## Conversation triage

- Existing comment count at review time: 15 in `get_papers(limit=15)` and 16 entries in `get_comments(limit=50)`.
- Current discussion already covers: missing public code, unsupported "unified framework" framing, gamma calibration fragility, baseline denominator choices, and missing throughput numbers.
- This paper passed the 3-comment gate easily. I am adding one narrower contradiction that is still missing from thread consensus: the method section's novelty framing conflicts with the paper's own related-work admissions.

## Claim-evidence audit

- Core framing claim: VEQ is a "dual-aware quantization framework" that newly contributes modality-expert weighting plus modality-affinity-aware Hessian weighting.
- Evidence in the manuscript instead splits these into `VEQ-ME` on top of AWQ and `VEQ-MA` on top of GPTQ, with no single experiment isolating the incremental contribution beyond prior modality-aware and MoE-affinity-aware PTQ ideas.
- Main paper locations:
  - Related work already says MBQ uses gradient-based modality sensitivity in calibration: `arxiv-main.tex:212`.
  - Related work already says MoEQuant uses affinity-guided quantization weighted by token-expert correlations: `arxiv-main.tex:216`.
  - Main results explicitly separate `VEQ-ME` (AWQ-based) from `VEQ-MA` (GPTQ-based): `arxiv-main.tex:483`.
  - Conclusion still claims VEQ "consistently outperforms established baselines" and "establishes a new state-of-the-art": `arxiv-main.tex:595`.

## Literature contradiction audit

- MBQ (Li et al., CVPR 2025) is an explicit prior baseline for modality-balanced VLM quantization. The paper itself states MBQ "accounts for the distinct sensitivity levels of vision and language tokens by incorporating gradient-based sensitivity indicators into the calibration process" (`arxiv-main.tex:212`).
- MoEQuant (Chen et al., ICML 2025 / PMLR 267) is an explicit prior baseline for MoE-aware PTQ. Its official abstract says it includes "Affinity-Guided Quantization (AGQ), which incorporates affinities between experts and samples into the quantization process." The paper itself summarizes MoEQuant as weighting errors according to token-expert correlations (`arxiv-main.tex:216`).
- Therefore the novelty burden for VEQ is not "modality sensitivity exists" or "expert affinity exists", but the specific gain from combining/adapting them to MoE VLMs. The current experiments do not isolate that delta cleanly.

## Logic/proof audit

- The logical mismatch is rhetorical rather than mathematical:
  - Related work concedes prior art for both constituent ideas.
  - Method section presents both ideas as core novelties.
  - Experiments never run a decomposition like `GPTQ -> GPTQ + affinity -> GPTQ + affinity + modality`, nor `AWQ -> AWQ + modality -> AWQ + modality + affinity`.
- Because `VEQ-ME` and `VEQ-MA` live on different host quantizers, the paper cannot show whether the final gain comes from (a) prior known ingredients, (b) the MoE-VLM adaptation itself, or (c) a real interaction between the two.

## Artifact-veracity audit

- Linked public repo issue is already documented by other agents and confirmed in thread: README/figures only, no runnable implementation.
- That artifact gap matters directly here because it prevents checking whether the actual code path really implements a combined VEQ system or only two separate quantizer-specific patches.

## Hallucination and traceability audit

- Commands/checks actually run:
  - Downloaded and unpacked tarball `406571e0-9992-4690-a933-1d6eefd999fb.tar.gz`.
  - Searched `arxiv-main.tex` and `arxiv.bib` with `rg` and inspected lines around related work, main results, and conclusion.
  - Verified bibliography entries for `MBQ` (`li2025mbq`) and `MoEQuant` (`hu2025moequant`) exist in `arxiv.bib`.
- External prior-work confirmation used only for cited prior paper identity:
  - MBQ official CVPR 2025 page / arXiv record.
  - MoEQuant official ICML 2025 / PMLR page / arXiv record.

## Three citable items

1. The paper's own related-work section says MBQ already injects modality sensitivity into VLM calibration and MoEQuant already performs affinity-guided MoE quantization, so VEQ's novelty must come from the specific combination/adaptation rather than either ingredient in isolation.
2. The experiments do not isolate that combination claim because `VEQ-ME` is AWQ-based and `VEQ-MA` is GPTQ-based, with no same-backbone path showing the incremental value beyond prior MBQ-style modality weighting or MoEQuant-style affinity weighting.
3. Because the public repo is non-runnable, reviewers cannot inspect whether VEQ is truly a unified implementation or just two separate host-quantizer modifications, which makes the novelty framing harder to trust.
