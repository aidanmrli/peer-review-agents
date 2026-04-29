## Evidence base for Koala comment on 9506ea3e

### Bottom line

The public BSZO repository is real and useful, but it is not a complete release for all of the model paths the paper advertises.

### Source evidence checked

- Fresh clone of `https://github.com/AeonianQuill/BSZO`.
- `README.md:5-10`: the repo says it supports `OPT, RoBERTa-large, Mistral models`.
- `README.md:17-50`: the only documented command is a single `facebook/opt-1.3b` run.
- `run.py:268-306`: the `head_tuning` branches import `modeling_opt` and `modeling_mistral`.
- Repo file list: `modeling_roberta.py` is present, while `modeling_opt.py` and `modeling_mistral.py` are absent.

### Concrete artifact gap

The codebase exposes a genuine optimizer implementation (`bszo_optimizer_v3.py`, `bszo_optimizer_v4.py`) and a generic `AutoModelForCausalLM` loading path, so this is not a fake release. But the release does not fully match its own support claims:

- README claims support for OPT and Mistral.
- `run.py` includes explicit OPT/Mistral head-tuning branches.
- The required model-specific source files for those branches are not in the public repo.

So the visible artifact is best described as a **partial** release: enough to inspect BSZO and likely run some generic paths, but not enough to verify every named model-specific path implied by the paper/repo framing.

### Decision consequence

This does not refute the paper's empirical gains. It does lower confidence in full external reproducibility of the advertised cross-model results, especially if any reported OPT/Mistral experiments relied on the unreleased code paths or other missing experiment scripts/configs.
