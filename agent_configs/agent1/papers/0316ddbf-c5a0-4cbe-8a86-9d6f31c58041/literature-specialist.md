# Literature Specialist Report

Paper: `0316ddbf-c5a0-4cbe-8a86-9d6f31c58041`, "Self-Attribution Bias: When AI Monitors Go Easy on Themselves"

Role: Literature Specialist for agent1. This report evaluates novelty, framing, and decision-relevant related work only. Empirical reproduction and implementation audit are left to the other role reports.

## Information Hygiene

Permitted sources used:

- Local paper source: `artifacts/sections/paper.tex`, especially lines 11-24, 31, 42-48, 75-100, 333-345, 362-364, and 406-424.
- Local bibliography: `artifacts/references.bib`, especially the entries for Panickssery et al. 2024, Wataoka et al. 2024, Spiliopoulou et al. 2025, Zheng et al. 2023, Wang et al. 2023, Chen et al. 2025, Tsui et al. 2025, Joglekar et al. 2025, Zhang et al. 2024, Yin et al. 2024, Lynch et al. 2025, Greenblatt et al. 2024, Hubinger et al. 2024, Huang et al. 2024, Self-Refine, Reflexion, and Constitutional AI.
- Koala metadata from `get_paper`: paper title, abstract, status, arXiv id, PDF/source artifact availability, and absence of GitHub URLs.
- Primary arXiv API records only for prior work: `2404.13076`, `2410.21819`, `2508.06709`, `2306.05685`, `2305.17926`, `2504.03846`, `2507.02778`, `2512.08093`, `2412.14470`, `2412.13178`, `2510.05179`, `2312.06942`, `2401.05566`, `1805.00899`, `2303.17651`, `2303.11366`, `2310.01798`, and `2212.08073`.

Forbidden sources not used: OpenReview reviews, scores, decisions, citation counts, social media, blogs, news coverage, or later reputation signals for this exact paper.

Commands/document access used:

```bash
curl -fsSL https://koala.science/skill.md
sed -n '1,260p' AGENTS.md
sed -n '1,260p' skills/literature-specialist.md
find papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041 -maxdepth 3 -type f | sort
rg -n "Self-Attribution|self-attribution|self preference|self-preference|LLM-as-judge|judge|self-correction|monitor|confession|agentic|related work|Related Work|references" papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041 -S
sed -n '1,620p' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/sections/paper.tex
sed -n '1,900p' papers/0316ddbf-c5a0-4cbe-8a86-9d6f31c58041/artifacts/references.bib
curl -fsSL https://export.arxiv.org/api/query?id_list=...
```

## Novelty Claim Checked

The paper's central literature claim is not that LLMs can prefer their own outputs in general. That is already established. The claimed contribution is narrower: the same action or output receives more favorable correctness or risk ratings when implicit conversational structure makes it look like the monitor's own current or prior action, with the strongest effect in on-policy self-monitoring and with static off-policy monitor evaluations therefore potentially overestimating deployment reliability.

That narrower claim is mostly supported as a distinct framing. The novelty is operational and safety-contextual: controlled attribution/turn-structure manipulations, on-policy versus off-policy action origin, and decision-relevant code/tool-use monitoring. It is not a completely new phenomenon in the broad sense of self-favoring LLM evaluation.

## Prior Work Considered

### Self-preference and self-bias in LLM-as-judge

Relevant prior work:

- Panickssery et al. 2024, "LLM Evaluators Recognize and Favor Their Own Generations."
- Wataoka et al. 2024, "Self-Preference Bias in LLM-as-a-Judge."
- Spiliopoulou et al. 2025, "Play Favorites: A Statistical Method to Measure Self-Bias in LLM-as-a-Judge."
- Chen et al. 2025, "Do LLM Evaluators Prefer Themselves for a Reason?"

The manuscript cites these works in the Related Work section and accurately distinguishes its target from ordinary self-preference. Panickssery et al. establish self-recognition/self-preference; Wataoka et al. argue familiarity or low perplexity can induce self-preference; Spiliopoulou et al. formalize and measure self-bias/family bias; Chen et al. is particularly close because it uses verifiable tasks and finds harmful self-preference when models are wrong. The present paper's strongest distinction is that it evaluates the same action under different attribution contexts, rather than choosing among different outputs or scoring distributions whose quality may differ.

This distinction is real but should be stated as an extension rather than an entirely independent phenomenon. The paper is strongest when it says prior self-preference work does not isolate implicit attribution and on-policy self-monitoring in safety-critical action evaluation. It would be too strong to imply that prior work did not already identify self-recognition, self-bias, and wrong-answer self-favoring as core problems.

### LLM-as-judge bias

Relevant prior work:

- Zheng et al. 2023, "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena."
- Wang et al. 2023, "Large Language Models are not Fair Evaluators."
- Stureborg et al. 2024 is in the bibliography but not used in the main related work.

The paper's framing is accurate: position, verbosity, self-enhancement, and ordering biases are established judge failures, and the current work studies a different source of evaluator unreliability. The paper's attribution manipulation is more targeted than generic judge bias, and the cross-model/off-policy comparisons are the right kind of evidence to separate self-attribution from prompt-wording artifacts.

Decision-relevant gap: the related-work paragraph is adequate for broad LLM-as-judge context. I would not mark the paper down materially for not expanding this section, because the key contribution is not another generic judge-bias benchmark.

### Self-correction and self-critique blind spots

Relevant prior work:

- Tsui et al. 2025, "Self-Correction Bench: Uncovering and Addressing the Self-Correction Blind Spot in Large Language Models."
- Huang et al. 2024, "Large Language Models Cannot Self-Correct Reasoning Yet."
- Self-Refine, Reflexion, and Constitutional AI as examples of systems that use the same model, or model-generated feedback, for critique/revision/evaluation.

Tsui et al. is the closest conceptual prior work in the paper's own framing: identical errors are corrected differently depending on whether they come from the model itself or from an external source. The manuscript cites this directly and fairly. Huang et al. is present in `references.bib` but is not cited in the main related-work text; it is not as close as Tsui et al. because it concerns intrinsic self-correction without external feedback rather than attribution-controlled evaluation, but it is decision-relevant background for the claim that self-monitoring/self-correction is unreliable.

The larger omission is not a single missing citation. The paper says "a common approach is self-monitoring" and gives examples of models critiquing or approving their own outputs, but the Related Work section has only a one-sentence "Self-monitoring" paragraph. Self-Refine, Reflexion, and Constitutional AI are important because they establish that self-feedback/self-critique is a major design pattern, even if their goals are improvement and alignment rather than risk monitoring. The paper does not need to benchmark against these methods, but it should position self-attribution bias against this lineage to avoid making the self-monitoring motivation look anecdotal.

### Self-monitoring and confessions

Relevant prior work:

- Joglekar et al. 2025, "Training LLMs for Honesty via Confessions."

The manuscript's one-sentence treatment is underdeveloped. Confessions are a very direct comparator: the same model is asked after an answer to report on its own compliance, shortcomings, or misbehavior. The distinction is still favorable to this paper: confessions explicitly train a separate honesty objective for the confession channel, while this manuscript evaluates naive monitor/risk/correctness judgments under attribution changes. That makes the present paper relevant as a warning about untrained or thresholded self-reporting monitors, not as a refutation of confession training.

Decision-relevant issue: because the paper reuses the agentic misalignment/confession ecosystem and argues about self-monitoring reliability, it should make this distinction explicit. Otherwise the Related Work section undersells the closest self-reporting prior work and leaves the reader to infer whether the result contradicts, complements, or bounds confession-style monitoring.

### Agentic safety benchmarks and tool-use safety

Relevant prior work:

- Zhang et al. 2024, "Agent-SafetyBench: Evaluating the Safety of LLM Agents."
- Yin et al. 2024, "SafeAgentBench: A Benchmark for Safe Task Planning of Embodied LLM Agents."
- Lynch et al. 2025, "Agentic Misalignment: How LLMs Could Be Insider Threats."
- Greenblatt et al. 2024, "AI Control: Improving Safety Despite Intentional Subversion."

The paper's agentic safety framing is broadly accurate. Agent-SafetyBench and SafeAgentBench evaluate safety behavior in interactive/tool-use settings, while this paper studies whether monitors correctly evaluate their own actions after generation. Lynch et al. supplies a high-risk agentic dataset that the paper reuses for monitor/risk assessment. Greenblatt et al. motivates architectural separation and monitoring under possible subversion, which makes the paper's "same model as actor and evaluator" concern directly relevant.

Missing citation/baseline: SafeAgentBench appears in the bibliography but is not cited in the main text. This is not fatal, but it would strengthen the claim that agentic safety benchmarks usually evaluate actions/agents rather than attribution-sensitive self-monitoring. I do not see a decision-critical missing experimental baseline from this literature; the correct baseline for the paper's claim is not another agent benchmark but attribution-controlled off-policy/cross-model judging, which the paper includes.

### Scheming/collusion framing

Relevant prior work:

- Hubinger et al. 2024, "Sleeper Agents."
- Greenblatt et al. 2024, "AI Control."
- Irving et al. 2018 and related scalable oversight work.

The paper's collusion framing is acceptable only because it explicitly says the studied phenomenon is unlikely to share the same roots as scheming-monitor collusion. The analogy is consequence-level, not mechanism-level: in both cases, evaluator and actor coupling can suppress risk signals. That is a defensible framing. It would become overclaimed if used to imply evidence about deception, scheming, or intentional subversion; the current text mostly avoids that.

## Overlap and Distinction

The overlap with prior work is substantial in three places:

1. Self-recognition and self-preference were already shown by Panickssery et al., Wataoka et al., Spiliopoulou et al., and Chen et al.
2. Attribution-sensitive self-correction was already shown directly by Tsui et al.
3. Self-monitoring/self-reporting designs were already central to Self-Refine, Reflexion, Constitutional AI, and confessions-style work.

The paper's distinction is still meaningful:

1. It manipulates implicit authorship by conversational structure rather than only explicit source labels or pairwise preferences.
2. It separates action origin from attribution, including on-policy versus off-policy settings and cross-model controls.
3. It moves the phenomenon into safety-relevant monitor decisions: code correctness, code risk, PR approval, computer-use risk, and agentic information leakage.
4. It ties the effect to a concrete evaluation failure mode: static off-policy monitor evaluations can look reliable even when on-policy self-monitoring is worse.

The literature therefore supports a moderate novelty claim: not "first evidence that LLMs favor themselves," but credible evidence for a specific, operationally important self-attribution failure mode in agentic self-monitoring.

## Missing Citation or Baseline

Decision-relevant missing or underdeveloped items:

- Huang et al. 2024 should be cited or discussed alongside Tsui et al. as broader evidence that intrinsic self-correction is unreliable. The paper already has the BibTeX entry, so this is an integration gap rather than a discovery gap.
- Self-Refine, Reflexion, and Constitutional AI should be mentioned as examples of self-feedback/self-critique pipelines. This would make the motivation for self-monitoring less anecdotal and help distinguish "self-monitoring for improvement" from "self-monitoring for safety gating."
- The confessions paragraph should be expanded. Confessions are close enough that the paper should explicitly say whether its findings challenge naive confession monitors, motivate trained confession objectives, or show a deployment hazard when the confession channel is evaluated off-policy.
- SafeAgentBench should be cited if the paper claims coverage of agentic safety benchmark context. Its omission is minor but avoidable.

I do not see a decisive missing baseline that invalidates the novelty framing. The existing cross-model/off-policy controls are the important literature-motivated baselines. A more exhaustive benchmark against Self-Refine/Reflexion would be a different paper and is not necessary for the central claim.

## Framing Accuracy

Mostly accurate, with one important caveat.

Accurate:

- The paper correctly says self-preference prior work studies models favoring own outputs or familiar/family outputs, while this work asks whether the same action is judged differently under implicit authorship attribution.
- The paper accurately connects LLM-as-judge bias to monitor unreliability but studies a narrower and more safety-relevant bias.
- The paper fairly treats self-correction blind spots as direct evidence that attribution can modulate evaluation.
- The agentic safety and AI-control connection is legitimate as a motivation for independent evaluation, provided the authors keep the mechanism separate from intentional scheming.

Caveat:

- The phrase "define and demonstrate self-attribution bias" is acceptable as a term/protocol contribution, but the underlying conceptual territory is already occupied by self-preference, self-recognition, harmful self-preference on verifiable tasks, and self-correction blind spots. The paper should frame novelty as "controlled implicit-attribution and on-policy self-monitoring in agentic safety settings," not as the first observation that models are self-favoring evaluators.

## Acceptance Consequence

The literature assessment is moderately positive but not decisive. Prior work substantially reduces the breadth of the novelty claim, especially because Chen et al. and Tsui et al. already show harmful self-preference and self-correction failures when models are wrong. However, the paper still contributes a meaningful operational extension: it isolates implicit self-attribution in conversational structure and studies the specific deployment hazard of using the same model as actor and monitor in agentic code/tool-use settings.

Score impact from literature alone: mild downgrade for incomplete self-monitoring/confessions framing and for not integrating broader self-correction/self-feedback work; no fatal novelty objection. If the empirical and artifact checks reproduce, the literature supports a weak-accept style contribution. If the empirical evidence is weak, prior work is too close for the novelty framing alone to rescue the paper.
