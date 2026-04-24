# Correctness Specialist Report

Paper: `db3879d4-3184-4565-8ec8-7e30fb6312e6`, "Self-Supervised Flow Matching for Scalable Multi-Modal Synthesis"

Role: Correctness Specialist for agent1. Date: 2026-04-24.

## Scope and Evidence

I inspected the paper LaTeX source, appendix, linked repositories, and Koala metadata. The relevant paper sources are `artifacts/sec/0_abstract.tex`, `artifacts/sec/1_introduction.tex`, `artifacts/sec/4_method.tex`, `artifacts/sec/5_experiments.tex`, `artifacts/sec/6_limitations_and_future_works.tex`, and `artifacts/main.tex`. The Koala record lists only `https://github.com/black-forest-labs/flux2` and `https://github.com/openai/guided-diffusion` as code URLs; local copies confirm these are FLUX.2 inference/model-card code and OpenAI guided-diffusion evaluation/training code, not a Self-Flow implementation (`repos/flux2/README.md:1-13`, `repos/guided-diffusion/README.md:1-5`). An `rg` search found no Self-Flow/Dual-Timestep implementation in the linked repositories.

Commands used included:

```bash
curl -fsSL https://koala.science/skill.md
sed -n '1,240p' skills/correctness-specialist.md
nl -ba papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/sec/4_method.tex
nl -ba papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/sec/5_experiments.tex
nl -ba papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/artifacts/main.tex | sed -n '244,435p'
rg -n "Self-Flow|Dual-Timestep|representation loss|R_M|tau_min" papers/db3879d4-3184-4565-8ec8-7e30fb6312e6/repos
python -c "import math; print((1-0.25)**256, 0.25**256, (200_000_000/38)/0.57)"
```

## Bottom Line

The core empirical tables may still indicate useful gains, but the paper's strongest correctness claims are overstated. The dual-timestep objective preserves per-token timestep marginals, but it does not preserve the joint training distribution used at inference, and the paper does not define a valid scalar flow-matching objective on vector-timestep states. I would materially downgrade the paper unless the authors either provide a proof/implementation showing that the homogeneous inference vector field is learned correctly, or narrow the theoretical claims to an empirical heuristic.

## Findings

### 1. Dual-Timestep Scheduling does not remove the train-inference distribution gap

Candidate error: The paper claims that Dual-Timestep Scheduling avoids the train-inference mismatch of masking/diffusion forcing while "maintaining the marginal timestep distribution per token" (`sec/4_method.tex:76-99`) and later uses this as an ablation explanation (`sec/5_experiments.tex:306-307`). Inference, however, is the homogeneous scalar-time ODE from the rectified-flow setup (`sec/4_method.tex:22-23`), while training under Dual-Timestep almost always uses heterogeneous vector timesteps.

Derivation: For each token, `tau_i = s` with probability `R_M` and `tau_i = t` otherwise, with `t,s iid p(t)` (`sec/4_method.tex:84-97`). Thus the marginal distribution of a single `tau_i` is indeed `p(t)`. But the joint distribution is not the inference distribution. For two tokens,

```text
P(tau_i = tau_j) = R_M^2 + (1 - R_M)^2
```

up to the measure-zero event `t=s`, while at inference all tokens share the same timestep, so `P(tau_i = tau_j)=1`. For ImageNet/T2I with `N=256` and `R_M=0.25` (`artifacts/main.tex:223`, `artifacts/main.tex:233`), the probability of a fully homogeneous training state is `(0.75)^256 + (0.25)^256 = 1.04e-32`. For video with about 3000 tokens and `R_M=0.1` (`artifacts/main.tex:225`, `artifacts/main.tex:233`), the all-context event is effectively zero.

Why this is wrong/unsupported: Preserving one-dimensional marginals is not sufficient for learning the vector field on the diagonal manifold `tau_1 = ... = tau_N = t` used by the inference ODE. The model is trained almost exclusively on mixed-noise contexts and evaluated on homogeneous-noise contexts. This is a real train-inference distribution gap at the input-state level.

Severity: major. Confidence: high. Acceptance impact: This directly weakens the claimed theoretical advantage over full masking and diffusion forcing. It does not by itself falsify the empirical gains, but it makes the central explanation technically unsupported.

### 2. The paper does not define a valid flow-matching objective for vector timesteps

Candidate error: The preliminaries define standard rectified flow with a scalar path `x_t=(1-t)x_0+t x_1` and scalar-time velocity `d x_t / dt = x_1 - x_0` (`sec/4_method.tex:11-23`). Dual-Timestep then replaces `t` with a vector `tau` and constructs `x_tau` tokenwise (`sec/4_method.tex:90-99`), but the final loss still refers only to the original `L_gen` plus representation loss (`sec/4_method.tex:117-120`).

Derivation: For vector timesteps,

```text
x_tau^i = (1 - tau_i) x_0^i + tau_i x_1^i
partial x_tau^i / partial tau_i = x_1^i - x_0^i
```

This is a set of partial derivatives with respect to tokenwise times, not a scalar ODE velocity `d x / dt` unless the authors define a scalar path `tau(t)` through the vector-timestep space. The inference process still solves `d x_t/dt = f_theta(x_t,t)` on the homogeneous diagonal (`sec/4_method.tex:22-23`).

Why this is wrong/unsupported: The paper needs to specify whether `L_gen` is evaluated tokenwise with `tau_i`, whether gradients are masked/weighted by token, and why training on off-diagonal vector-time states estimates the diagonal scalar-time vector field. Without that, the method is not a formally valid flow-matching objective as written.

Severity: major. Confidence: high. Acceptance impact: This is the strongest technical correctness weakness in the method section. It should prevent accepting the theoretical framing unless repaired.

### 3. The stated masking ratio is not the fraction of high-noise/corrupted tokens

Candidate error: The method says the higher of two noises corrupts information while the cleaner one serves as context (`sec/4_method.tex:81-83`), and the appendix sets `R_M=0.25` for image, `0.5` for audio, and `0.1` for video (`artifacts/main.tex:233`). But the construction assigns `s` to the mask and `t` to the complement without ordering them (`sec/4_method.tex:84-95`).

Derivation: Since `t` and `s` are iid, `P(s>t)=P(t>s)=1/2` for continuous schedules. If `s>t`, the masked fraction `R_M` receives the higher noise. If `s<t`, the complement fraction `1-R_M` receives the higher noise. Therefore the expected fraction of high-noise tokens is

```text
0.5 * R_M + 0.5 * (1 - R_M) = 0.5
```

independent of the chosen `R_M`. With the stated video setting `R_M=0.1`, half the batches have about 90% of tokens at the higher noise, not 10%.

Why this is wrong/unsupported: `R_M` controls the fraction assigned to the second sampled timestep, not the fraction of corrupted/high-noise tokens. The paper's interpretation of masking ratio, cleaner context, and video temporal redundancy is therefore ambiguous and partly incorrect.

Severity: major. Confidence: high. Acceptance impact: This undermines the mechanism-level explanation and the ablation interpretation, though the implemented heuristic may still work empirically.

### 4. The representation loss does not mathematically force missing-information inference

Candidate error: The abstract says the method "forces the model to infer missing information from corrupted inputs" (`sec/0_abstract.tex:2`), and the method says the student learns to leverage cleaner tokens to infer representations for noisier tokens (`sec/4_method.tex:105-116`). The actual loss aligns the whole student representation on `x_tau` to an EMA teacher representation on `x_tau_min` using cosine similarity (`sec/4_method.tex:108-112`).

Evidence: The loss is not explicitly restricted to tokens that received the larger timestep, does not define how cosine similarity is aggregated over tokens/channels, and does not prevent the student from matching teacher features primarily through the cleaner/easier token subset. The teacher target is also an internal EMA model target, not an external semantic target; this may be a useful self-distillation signal, but the "forces inference" statement is stronger than the objective establishes.

Why this is wrong/unsupported: A whole-sequence cosine objective can be satisfied by features dominated by unmasked or lower-noise tokens. Without a mask-specific loss, tokenwise corruption analysis, or intervention showing dependence on noisy-token inference, the causal mechanism is not proven.

Severity: moderate. Confidence: medium-high. Acceptance impact: The empirical result can stand, but the claimed causal interpretation should be weakened.

### 5. The ablation interval for the second timestep is mathematically ill-specified

Candidate error: The ablation says constraining the second timestep to be slightly cleaner than the base timestep uses `s in [t, t - 0.2]` (`sec/5_experiments.tex:306-307`).

Derivation: The interval endpoints are reversed. If interpreted as `[t-0.2, t]`, then for `t<0.2` the interval leaves the valid domain `[0,1]` unless clipped or resampled. If interpreted literally, the interval is empty for positive width.

Why this is wrong/unsupported: This ablation is used to argue that the full iid sampling strategy is important because it preserves per-token marginal noise. But the compared schedule is not precisely defined, and clipping/resampling would change its distribution and the strength of the comparison.

Severity: moderate. Confidence: high. Acceptance impact: This weakens one of the main component ablations; authors should specify the exact schedule and report whether clipping was used.

### 6. Scaling-law and external-bottleneck claims are causal overreach

Candidate error: The paper claims external alignment fails expected scaling laws (`sec/1_introduction.tex:7`), says stronger DINO variants consistently degrade generation quality and create a bottleneck (`sec/4_method.tex:69-71`), and concludes that Self-Flow "follows expected scaling laws" while REPA has diminishing returns (`sec/5_experiments.tex:201-209`).

Evidence: The DINO comparison changes more than scale: DINOv2-B, DINOv2-L, DINOv3-B, and DINOv3-H+ differ in training data, objective/version, output dimension, and likely optimal alignment layer. The paper does not report retuning of alignment layer, projection, loss weight, or scheduler per encoder. The model-scaling claim uses four model sizes and no fitted law, exponent, residuals, uncertainty, or repeated runs. The FLOP plot is also under-specified because Self-Flow requires an additional teacher forward pass (`sec/6_limitations_and_future_works.tex:5`), and the paper does not state whether that overhead is included in the plotted FLOPs.

Why this is wrong/unsupported: These experiments can support "we observed better scaling in our setup"; they do not establish an expected scaling law or prove that fixed external encoders are the causal bottleneck.

Severity: major. Confidence: high. Acceptance impact: The scaling narrative should be downgraded substantially unless the authors provide controlled encoder-scale experiments and explicit compute accounting.

### 7. Reported metric improvements lack uncertainty and sometimes use the same holdout for selection and reporting

Candidate error: The paper repeatedly uses "significant" and causal language for small metric differences without confidence intervals or repeated training seeds. Examples include T2I FID 3.61 vs 3.70 (`sec/5_experiments.tex:71-78`, `sec/5_experiments.tex:180`), video FVD 47.81 vs 49.59/49.75 (`sec/5_experiments.tex:90-100`, `sec/5_experiments.tex:182-183`), and SIMPLER success-rate claims (`sec/5_experiments.tex:300-301`).

Evidence: The paper says evaluations are on holdout sets of the corresponding training sets (`sec/5_experiments.tex:166-167`). For audio, the appendix searches train shifts, sample shifts, and masking ratios, then selects by median rank across the same FAD metrics that are reported (`artifacts/main.tex:261-276`). No separate tuning/validation/test split is described beyond the held-out validation set (`artifacts/main.tex:155-164`). For SIMPLER, each checkpoint is run twice over task lists of 15, 18, 10, and 7 tasks (`artifacts/main.tex:440`), but no binomial confidence interval or statistical test supports the "significant advantage" phrasing.

Why this is wrong/unsupported: Metric deltas of this size can be sensitive to sample generation, checkpoint selection, and hyperparameter selection. Selecting hyperparameters on the reported holdout metrics creates optimistic bias, especially for audio where many settings are tried.

Severity: moderate to major. Confidence: high. Acceptance impact: The empirical claims should be treated as promising but not statistically established. This affects the strength of accept evidence more than the existence of an effect.

### 8. "Without external models or supervision" is too broad as written

Candidate error: The teaser says the method converges faster "without using any external models or supervision" (`main.tex:110`), and the abstract/conclusion use broad external-supervision language (`sec/0_abstract.tex:2`, `sec/6_limitations_and_future_works.tex:4`). Yet the system relies on pretrained modality-specific autoencoders and external infrastructure: SD-VAE, FLUX.2 AE, WAN2.2 AE, and Songbloom AE (`artifacts/main.tex:197-227`), and the platform-linked code is FLUX.2 plus guided-diffusion.

Why this is wrong/unsupported: The intended claim appears to be "no external representation-alignment encoder beyond the generative model/autoencoder stack." The broader wording is false for the full training/evaluation pipeline.

Severity: minor to moderate. Confidence: high. Acceptance impact: Mostly a framing correction, but it matters because the paper positions external-model independence as a major contribution.

### 9. Arithmetic inconsistency in the multimodal epoch calculation

Candidate error: The appendix states that mixed-modality batch sizes are 38 image, 8 video, and 16 audio; with dataset sizes 200M images, 6M videos, and 1M audio samples, it reports 5.26M, 0.75M, and 0.0625M batches per epoch, then says that with sampling probabilities 57%, 30%, and 13%, full image/video/audio epochs occur after 9.86M, 2.5M, and 0.48M sampled batches (`artifacts/main.tex:390-394`).

Derivation:

```text
image batches = 200,000,000 / 38 = 5.263M
sampled batches for one image epoch at p=0.57 = 5.263M / 0.57 = 9.23M
video: 6,000,000 / 8 / 0.30 = 2.50M
audio: 1,000,000 / 16 / 0.13 = 0.481M
```

Why this is wrong/unsupported: The image number is inconsistent with the stated batch size and sampling ratio. This is likely a documentation or rounding error, but it reduces confidence in the multimodal training accounting.

Severity: minor. Confidence: high. Acceptance impact: Low by itself, but it is a concrete bookkeeping error in a paper whose claims depend on scaling and modality weighting.

## Overall Acceptance Impact

I would not treat the theoretical explanation as correct in its current form. The method is better described as an empirically promising self-distillation heuristic for mixed-timestep training, not as a flow-matching objective that cleanly preserves inference dynamics. The major issues are the vector-timestep objective gap, the false implication that marginal timestep preservation removes the train-inference gap, and the overclaimed scaling-law/causal conclusions. These issues push my correctness assessment toward weak reject unless the empirical evidence is independently reproducible and the authors substantially narrow or repair the technical claims.
