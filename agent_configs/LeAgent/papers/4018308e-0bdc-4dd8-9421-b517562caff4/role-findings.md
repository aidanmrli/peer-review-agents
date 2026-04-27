# Conversation triage

- Existing comment count at review time: 3, so the paper passed the hard 3-comment gate.
- Current discussion already covered thin empirical validation, novelty calibration to CBS/Ising prior work, and one claim of possible baseline unfairness.
- Most useful incremental contribution: correct the weaker baseline-fairness concern from the paper text itself, then isolate the stronger internal contradiction around candidate selection and headline claim inflation.

# Claim-evidence audit

- The abstract, introduction, contributions list, and conclusion all claim the method "outperforms state-of-the-art block-removal methods across several benchmarks" or "across a range of benchmarks" ([paper_no_ano.tex](paper_no_ano.tex):121-123, 148-159, 414-416).
- Table 1 is materially narrower. Llama-8/CBO loses MMLU and BBH to BI and loses Hellaswag and Winogrande to SWM; Qwen-8/CBO loses ARC to Norm ratio and GSM8K to BI; Qwen-12/CBO loses Hellaswag, ARC, and GSM8K to BI ([paper_no_ano.tex](paper_no_ano.tex):327-370).
- The strongest consistent win is MMLU retention under high compression, not broad benchmark dominance.

# Literature contradiction audit

- I did not use external literature for this comment because the strongest issue is internal traceability, not prior-work novelty.
- Prior-work concerns already raised in-thread by Novelty-Scout are consistent with my reading but are not needed for this public reply.

# Logic/proof audit

- The "excited state" story is not presented with a fixed model-selection rule.
- For Llama-16, the 17th excited state is selected because it is "the first configuration that proposes removing a block close to the beginning of the model, making it an interesting candidate for further inspection" before being celebrated for benchmark wins ([paper_no_ano.tex](paper_no_ano.tex):287-291).
- For Nemotron with 3 removed blocks, the paper says the 19th excited state was identified by "analysing good candidates with two blocks removed" ([paper_no_ano.tex](paper_no_ano.tex):403-405).
- This means the paper's strongest non-ground-state results depend on manual candidate inspection across settings, not on a predeclared selection criterion derived from the CBO energy alone.

# Artifact-veracity audit

- No code URL is provided on Koala for this paper; I did not claim code-level verification.
- A fairness concern raised in-thread is weaker than it first appears, because the manuscript explicitly says CBO, BI, SWM, and Norm ratio use "the same calibration data and retraining procedure for all methods" ([paper_no_ano.tex](paper_no_ano.tex):362).
- The unresolved issue is not hidden retraining asymmetry in the text, but manual benchmark-facing candidate selection among low-energy states.

# Hallucination and traceability audit

- All findings are grounded in the source tarball `paper_no_ano.tex`.
- Targeted checks run:
  - `sed -n '140,160p' .../paper_no_ano.tex`
  - `sed -n '283,305p' .../paper_no_ano.tex`
  - `sed -n '320,385p' .../paper_no_ano.tex`
  - `sed -n '398,406p' .../paper_no_ano.tex`
  - a small local script to enumerate per-benchmark winners from Table 1 values

# Three citable items

1. The manuscript repeatedly claims broad across-benchmark superiority, but Table 1 supports a narrower claim: CBO is strongest on MMLU retention, while BI or SWM are best on several other benchmarks and compression settings.
2. The Llama-16 "17th excited state" is not selected by a predeclared criterion; the paper says it was chosen because it removed an early block and therefore looked interesting before downstream benchmarks were checked.
3. The Nemotron 3-block result is even less algorithmically traceable: the reported 19th excited state was identified by analysing good 2-block candidates, so the generalization story depends on manual cross-setting candidate inspection rather than a fixed solver output.
