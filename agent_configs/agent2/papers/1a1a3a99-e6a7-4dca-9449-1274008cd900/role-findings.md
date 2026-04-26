# TIC-VLA Role Findings

## Reproducibility lead: central claim and reproduction target
Central claim: TIC-VLA enables robust language-guided robot navigation under multi-second reasoning latency via a delayed semantic-control interface, latency-consistent training, and the new DynaNav benchmark. Reproduction target: recover the DynaNav simulation results, latency ablations, and real-robot success-rate table from released artifacts.

## Reproducer A: artifact-first check
The Koala tarball contains only paper source and figures: `main.tex`, `supp.tex`, PDFs, bibliography, and style files. No simulator scenes, Isaac Sim / Isaac Lab code, asynchronous controller code, training scripts, configs, checkpoints, data manifests, or evaluation scripts are included. The project website links `https://github.com/ucla-mobility/TIC-VLA`; cloning that repository on 2026-04-26 yields only `README.md` plus website assets under `docs/`. The README explicitly marks the dataset as "Coming Soon."

## Reproducer B: clean-room/specification check
From the paper source alone I can reconstruct high-level methodology but not the runnable system. Load-bearing unreleased components include: DynaNav's 85 benchmark episodes across four Isaac Sim environments; the custom human behavior controller; teleoperation and data-preprocessing pipeline for 310 episodes; GPT-5 prompting/annotation pipeline for instructions and reasoning traces; asynchronous `predict_async` / KV-cache execution logic; PPO training configuration for delayed-inference RL; and real-robot deployment stack for Unitree Go2 on Orin NX / RTX 4060 / A6000.

## Implementation auditor: code/artifact/repo match
The paper and website present TIC-VLA and DynaNav as implemented systems, but the public repo is website-only and the tarball is manuscript-only. This is a release mismatch, not a partial implementation release. I found no public code for Isaac Sim scenes, policy modules, VLM fine-tuning, RL, inference scheduling, controller smoothing, or benchmark evaluation.

## Correctness specialist: methods, metrics, proofs, or conclusion risks
The strongest empirical claims depend on unreleased infrastructure. Real-world results are averaged over five trials per task, which already makes variance hard to assess; without code/logs/videos/configs, the latency-robustness and edge-deployment claims cannot be independently checked. The paper also relies on GPT-5-generated instructions/reasoning for supervision, but prompt templates and generated annotations are not released.

## Literature specialist: novelty/framing against permitted prior work
The latency-aware control framing is plausible and relevant to embodied VLA deployment. My main concern is not novelty but validation: the benchmark, data-generation pipeline, and deployment stack are core contributions and currently unavailable for audit.
