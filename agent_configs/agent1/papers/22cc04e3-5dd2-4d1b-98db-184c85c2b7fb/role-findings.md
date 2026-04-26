## Reproducibility lead
Central claim and reproduction target: the released artifact should implement the VETime training/evaluation recipe closely enough to support the paper's "strictly zero-shot" headline results on TSB-AD and the Appendix C4 implementation details.

## Reproducer A
Artifact-first check: cloned `https://github.com/yyyangcoder/VETime`, inspected `README.md`, `train.py`, `Test_TSB.py`, `model/Vision_encoder/V_encoder.py`, `model/TS_encoder/*`, and the Koala source tarball. The repo is real and nontrivial, but it does not match the paper's stated training recipe. `train.py` uses `torch.optim.Adam(..., lr=1e-4, weight_decay=1e-2)` and parser default `--num_epochs 4`, while the paper says AdamW, lr `5e-4`, weight decay `1e-5`, up to 25 epochs with early stopping.

## Reproducer B
Clean-room/specification check: Appendix C4 says the temporal encoder is adapted with LoRA rank 8 and alpha 16, with the base weights frozen. I searched the release for `lora`, `peft`, `get_peft_model`, and adapter wiring. The only hits are `peft>=0.4.0` in `requirements.txt` and an unused `from peft import LoraConfig` import in `model/Vision_encoder/V_encoder.py`. I did not find actual LoRA insertion or PEFT wrapping in either the time-series encoder or the vision encoder.

## Implementation auditor
Code/artifact/repo match: the repo contains core modules for reversible image conversion, alignment, fusion, loss, and evaluation. However, the released implementation differs materially from the written spec:
- Paper Appendix C4: LoRA on time-series encoder, AdamW, lr `5e-4`, wd `1e-5`, 25 epochs.
- Repo: no LoRA application found; `train.py` uses plain Adam, lr `1e-4`, wd `1e-2`, default 4 epochs.
- The paper frames VETime as "strictly zero-shot", but the appendix and code train end-to-end with anomaly labels (`L_BCE` in the paper; `model.anomaly_detection_loss(..., labels)` and label-conditioned anomaly-window contrastive logic in code).

## Correctness specialist
Methods/metrics/conclusion risks: the mismatch is not a cosmetic documentation issue. If the released code and defaults are what produced the checkpoint, then the appendix is inaccurate. If the appendix is accurate, the public repo is insufficient to reproduce the reported system. Either way, the current artifact weakens confidence in the exact source of the reported gains and in the fairness of the "strictly zero-shot" comparison language.

## Literature specialist
Novelty/framing against permitted prior work: the synthetic-data pretraining plus labeled anomaly objective seems closer to "no target-domain finetuning" than to unsupervised or task-agnostic zero-shot. That framing distinction matters because several compared baselines are forecasting-style foundation models without anomaly-supervised pretraining.
