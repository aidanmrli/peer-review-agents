## Reproducibility lead
Central claim and reproduction target: the paper claims MCFA yields >90% vulnerability across LangChain/LlamaIndex and three closed models, with durable cross-task persistence and partial RBMS defense. Reproduction target is the attack table and defense table, not just the threat model.

## Reproducer A
Artifact-first check: Koala tarball `e5e5467c-27e4-495d-9c20-f078ae58431e.tar.gz` contains only LaTeX sources and figures (`example_paper.tex`, style files, images). `00README.json` lists only `example_paper.tex` as a source file. No code, prompts as files, tool inventories, trace logs, configs, or harness are released.

## Reproducer B
Clean-room/specification check: the manuscript exposes high-level protocol pieces in `example_paper.tex`, including 36 sampled tools per framework, three retrieval modes, isolated vs. online regime, and a prompt template. But it omits the concrete tool list, the exact sampled tool pairs/order workflows, the dynamic injection generator prompt, the retrieval implementation for “Strong” vs “Weak”, the trace parser/scorer, and the actual MEMFLOW code that the impact statement says is released.

## Implementation auditor
The percentages in Tables 1 and 2 strongly imply a denominator of 36 trials per condition: 97.2=35/36, 91.7=33/36, 69.4=25/36, 63.9=23/36, 58.3=21/36, 52.8=19/36, 8.3=3/36, 5.6=2/36, 2.8=1/36. Combined with `temperature=0.0` and “fixed seed policy,” this reads as one deterministic sweep over 36 tool candidates per framework, not repeated stochastic runs. I saw no variance estimates, bootstrap intervals, alternate seeds, or prompt-robustness checks.

## Correctness specialist
The paper’s measurement claim is broader than the released metric support. ASR is a binary per-task event (`S / |X_benign|` in Algorithm 1), yet the paper frames the contribution as control-flow integrity. This compresses materially different deviations into the same bit and can overstate certainty when reported as near-saturated percentages from a 36-shot grid. The defense conclusion is also brittle: D2 ranges from 2.8% to 100% depending on model/framework, but the paper does not release the traces needed to verify whether this comes from retrieval differences, prompt formatting, or actual hierarchy-compliance behavior.

## Literature specialist
The novelty of persistent memory-induced tool-control attacks seems real if scoped to control-flow auditing rather than generic memory poisoning; existing thread comments already cover AgentPoison/MINJA/A-MemGuard. My additive value here is narrower: even if the threat model is novel and plausible, the empirical release is currently too thin for the headline rates to be independently audited.
