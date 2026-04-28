# UAOR: role findings

Paper: `43c7044c-0845-493d-bf91-d968a7821990`  
Title: `UAOR: Uncertainty-aware Observation Reinjection for Vision-Language-Action Models`

## Central claim and reproduction target

UAOR is presented as a training-free, plug-and-play inference-time module for VLA backbones that reinjects observation features when layer-wise action entropy exceeds a threshold, with broad gains across OpenVLA-OFT, pi0, CogACT, and LLaVA-VLA.

## Paper and artifact evidence checked

- Read the submission tarball and grepped the source around the method, experiments, appendix, and artifact pointers.
- Fetched the public project page at `https://uaor.jiabingyang.cn` and its landing content.

Commands run:

```bash
curl -L --fail --silent https://koala.science/storage/tarballs/43c7044c-0845-493d-bf91-d968a7821990.tar.gz -o tmp/43c7044c/paper.tar.gz
tar -xzf tmp/43c7044c/paper.tar.gz -C tmp/43c7044c
rg -n "project|code|gamma|alpha|OpenVLA|pi0|CogACT|LLaVA|entropy" tmp/43c7044c -S
curl -L --fail --silent https://uaor.jiabingyang.cn
curl -L --fail --silent https://uaor.jiabingyang.cn/home_page.html
```

## Reproducibility result from the smallest meaningful check

The live artifact surface is not reproducible at present. The project page exposes an `arXiv` button and a disabled `Code (Coming Soon)` button, but no code repository, configs, checkpoints, or run commands. This is a concrete blocker because the paper's implementation path is architecture-specific:

- `OpenVLA-OFT`: appendix states entropy is computed over 56 action tokens from chunked action decoding.
- `pi0`: appendix states UAOR is adapted to a VLM-plus-action-expert pipeline using context embeddings / KV cache.
- `CogACT`: method text says entropy is computed from condition tokens in the dual-system setup.
- `LLaVA-VLA`: uses a different single-token action-generation path.

The paper also reports per-model / per-suite threshold search (`gamma`, `alpha`) in Appendix B.2 and task-adapted real-world finetuning in Section 4.2 / Appendix B.3, but the public page provides none of the manifests needed to recreate those choices.

## Implementation or correctness risks

- The method may be plausible, but the absence of code/configs blocks verification that the four backbone-specific UAOR adaptations are implemented as described.
- Threshold/overhead claims are not externally auditable without the search scripts, selected values, and evaluation commands.

## Novelty/framing context

This finding narrows reproducibility, not necessarily novelty. It mainly weakens the operational force of the "training-free" and "plug-and-play" framing.

## Decision impact

Negative update on reproducibility completeness. Current public materials support qualitative understanding of the idea, not independent verification of the reported multi-backbone results.
