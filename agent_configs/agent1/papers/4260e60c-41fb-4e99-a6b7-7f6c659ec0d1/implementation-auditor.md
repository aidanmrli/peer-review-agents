# Implementation Auditor Report

Paper: `4260e60c-41fb-4e99-a6b7-7f6c659ec0d1`, "Demystifying When Pruning Works via Representation Hierarchies"  
Repository inspected: `repos/Pruning-on-Representations` at commit `ba1c25bd4d08c27319bf85d5a75b381fa70279d6`  
Audit scope: static artifact/code inspection only. I did not install dependencies or run model inference.

## Claim Being Tested

The paper claims a reproducible empirical and mechanistic explanation for when pruning works: non-generative performance is often preserved while generation collapses, and this is explained through representation shifts across hidden, logit, and probability spaces. The paper states that code is available at `https://github.com/CASE-Lab-UMD/Pruning-on-Representations` (`artifacts/main.tex:70`) and reports benchmark, pruning, and representation-analysis settings in `artifacts/sections/appendix.tex:8-18`.

## Commands Run

```bash
curl -fsSL https://koala.science/skill.md | sed -n '1,220p'
sed -n '1,220p' skills/implementation-auditor.md
git -C papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations rev-parse HEAD
git -C papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1/repos/Pruning-on-Representations status --short
rg --files papers/4260e60c-41fb-4e99-a6b7-7f6c659ec0d1
rg -n "(HumanEval|MBPP|NarrativeQA|NQ-Open|BEIR|GSM8K|hellaswag|lm_eval|eval_zero_shot)" --glob '*.py' --glob '*.sh' --glob '*.md' .
find . -path './.git' -prune -o -path './inter-layer/src/llmtuner/data/c4_train.json' -prune -o -path './inter-layer/src/llmtuner/data/c4_val.json' -prune -o -path './inter-layer/src/llmtuner/data/c4_demo.json' -prune -o -type f \( -name '*.csv' -o -name '*.jsonl' -o -name '*result*' -o -name '*log*' -o -name '*.out' -o -name '*.pt' -o -name '*.npy' \) -print
```

The nested repository was clean. The only unrelated outer worktree change observed was `../../config.toml`.

## Artifact Inventory

- Paper source/PDF and static figures are present under `artifacts/`.
- The author repo contains `README.md`, root `requirements.txt`, `intra-layer/`, `inter-layer/`, `representation-analysis/`, static `figs/`, and vendored quantization code.
- I found no raw benchmark result tables, CSVs, JSON outputs, run logs, model checkpoint metadata, similarity caches, or plotting notebooks/scripts that regenerate the paper figures. Static `figs/` and `docs/figs/` are present, but the command above found no result artifacts beyond code, static figures, and C4 sample JSON.

## Code Paths Inspected

- Public run instructions: `README.md:49-59`, `README.md:106-132`, `README.md:199-221`, `README.md:242-259`.
- Paper experimental claims: `artifacts/sections/appendix.tex:8-18`, `artifacts/sections/experiments.tex:4-18`, `artifacts/sections/experiments.tex:60-80`, `artifacts/sections/experiments.tex:97-105`, `artifacts/sections/experiments.tex:220-260`.
- Intra-layer pruning: `intra-layer/main.py`, `intra-layer/lib/data.py`, `intra-layer/scripts/prune.sh`.
- Inter-layer dropping: `inter-layer/scripts/dropping/*.sh`, `inter-layer/src/llmtuner/compression/prune/workflow.py`, `layer_drop.py`, `block_drop.py`, `utils.py`, `io.py`.
- Evaluation/speed scripts: `inter-layer/scripts/benchmark/benchmark_lm_eval.sh`, `inter-layer/scripts/benchmark/benchmark_speed.sh`, `inter-layer/src/benchmark_speed.py`.
- Representation analysis: `representation-analysis/transition_layerwise_compare.py`, `compare_generation_metrics.py`, `compare_mcq_subspace_metrics.py`, `generation_forward_utils.py`, `transition_metrics_logging.py`.
- Qwen custom model artifact: `modeling_qwen.py`.

## Paper-to-Code Matches

- The repo does contain Wanda/SparseGPT pruning entry points matching the paper's intra-layer methods: `intra-layer/main.py:36-51` exposes `--prune_method`, `--sparsity_ratio`, `--sparsity_type`, `--seed`, `--nsamples`, and `--save_model`; `intra-layer/lib/data.py:40-83` samples C4 sequences, consistent with the paper's C4 calibration description in `appendix.tex:11-15`.
- Inter-layer layer/block dropping code exists. `inter-layer/src/llmtuner/compression/prune/workflow.py:102-114` dispatches `layer_drop`/`block_drop` and saves reserved-layer configs; `layer_drop.py:144-162` selects high-similarity layers; `block_drop.py:222-237` selects discrete block drops.
- Representation metrics are implemented at least in skeleton form. `transition_metrics_logging.py:40-140` computes hidden/logit cosine, vocabulary cosine, KL, and second-order variance estimates, matching the paper's representation hierarchy at a high level.

## Paper-to-Code Discrepancies and Bugs

1. The public artifact does not reproduce the paper's reported benchmark suite.
   - The paper reports GSM8K, HumanEval, MBPP, NarrativeQA, NQ-Open, multiple MCQ tasks, and BEIR retrieval (`appendix.tex:17-18`), plus Qwen HellaSwag/GSM8K Wanda results (`experiments.tex:4-8`).
   - The only benchmark script I found is `inter-layer/scripts/benchmark/benchmark_lm_eval.sh:11-39`; it evaluates BoolQ/RTE/OpenBookQA/PIQA/MMLU/WinoGrande/GSM8K/HellaSwag/ARC-Challenge for a single Mistral example, but not HumanEval, MBPP, NarrativeQA, NQ-Open, or BEIR retrieval. The intra-layer path has an optional zero-shot list only (`intra-layer/main.py:117-127`) and explicitly skips evaluation if `lib.eval` is missing (`intra-layer/main.py:10-15`, `intra-layer/main.py:102-120`). I found no `lib/eval.py` in the repo.
   - Decision consequence: the main reported performance discrepancy cannot be independently recomputed from the released artifact.

2. The default Qwen inter-layer story is not supported by the released inter-layer dropping pipeline.
   - The paper says Qwen-2.5-7B-Instruct is the main model (`appendix.tex:8-9`) and Figure 2/analysis use Qwen layer dropping (`experiments.tex:73-105`, `experiments.tex:220-249`). The README also uses Qwen in dropped-mode commands (`README.md:110-117`, `README.md:203-210`, `README.md:247-253`).
   - The inter-layer post-drop saver supports only `llama`, `mistral`, `deepseek`, `gemma2`, and `baichuan` in `auto_map`/`CUSTOM_FILE` (`inter-layer/src/llmtuner/compression/prune/utils.py:142-196`). `layer_drop.py:170-175` and `block_drop.py:246-251` raise `Unsupported model type!` otherwise. There is a root `modeling_qwen.py`, but no `configuration_qwen2.py` exists; `modeling_qwen.py:34-35` imports that missing relative file and `transition_metrics_logging`, so it is not a self-contained dropped-Qwen module.
   - Decision consequence: the main-model Qwen inter-layer dropped checkpoints are not reproducible from the released inter-layer code.

3. The README dropped-analysis commands are likely no-ops unless the user already has a custom dropped model object.
   - `representation-analysis/generation_forward_utils.py:94-103` only sets `drop_attn`/`drop_mlp` when those attributes already exist on each layer.
   - `transition_layerwise_compare.py:236` loads `AutoModelForCausalLM.from_pretrained(model_name)` and in dropped mode reuses that dense model (`transition_layerwise_compare.py:290-312`); `compare_generation_metrics.py:195-244` follows the same pattern. The README example passes `Qwen/Qwen2.5-7B-Instruct` (`README.md:114`, `README.md:207`, `README.md:250`), whose standard HF layers do not expose repo-specific `drop_attn`/`drop_mlp` flags.
   - Decision consequence: the public commands can silently compare dense-vs-dense and log misleading "dropped" metrics.

4. The counterfactual pruned-layer attention hook does not preserve autoregressive context.
   - The paper says it replaces only the current layer while keeping all other layers and the current dense context fixed (`experiments.tex:76-80`).
   - In pruned mode, the hook calls the pruned attention with `past_key_value=None`, `use_cache=False`, and `cache_position=None` (`transition_layerwise_compare.py:67-77`) even though generation uses `use_cache=True` (`transition_layerwise_compare.py:227`, `generation_forward_utils.py:164-193`). After the prefill step, this pruned attention call sees only the current token hidden input without the dense historical KV cache. That is not the stated shared-context intervention.
   - Decision consequence: the layerwise pruned-mode attention metrics after the first step are not faithful to the paper's described intervention.

5. The representation-analysis scripts use one hardcoded toy prompt rather than the paper's "multiple prompts" or benchmarks.
   - `transition_layerwise_compare.py:244-246` and `compare_generation_metrics.py:197-199` use a single "John has twice as many books..." prompt. `compare_mcq_subspace_metrics.py:118-125` uses a single "Which animal is a mammal?" prompt and hardcodes lowercase answer tokens `" a"..." d"` at `compare_mcq_subspace_metrics.py:198-205`.
   - The paper claims layerwise figures are measured over multiple prompts (`experiments.tex:104`) and gives a broader prompt table (`appendix.tex:816-867`).
   - Decision consequence: the released scripts are closer to demos than reproduction scripts for the plotted results.

6. Calibration details are inconsistent and partly hardcoded.
   - The paper says pruning masks use 128 C4 samples (`appendix.tex:11-15`). Intra-layer defaults match this (`intra-layer/main.py:38-39`), but inter-layer scripts use 256 samples in `layer_drop.sh:5-9` and `block_drop.sh:5-8`; only `layer_drop_iterative.sh:5-8` uses 128.
   - `intra-layer/scripts/prune.sh:5-8` defaults to Mistral with placeholder `your_model_root_path` and `your_model_output_path`, while the paper's default Figure 1/3 Qwen setting is only commented out at `prune.sh:3`.
   - Decision consequence: reproducing the exact reported Qwen settings requires undocumented edits.

7. Dependencies are incomplete for the code paths advertised.
   - Root `requirements.txt` pins only PyTorch/HF basics. It omits packages imported by repo code, including `peft`, `trl`, `lm_eval`, `auto_gptq`, `awq`, and `matplotlib`/plotting dependencies used or required by inter-layer/benchmark utilities (`inter-layer/src/llmtuner/model/loader.py:1`, `inter-layer/src/llmtuner/model/adapter.py:4`, `inter-layer/scripts/benchmark/benchmark_lm_eval.sh:33-39`, `inter-layer/src/benchmark_speed.py:11-12`).
   - `inter-layer/setup.py:14-21` looks for `inter-layer/requirements.txt`, but no such file exists.
   - Decision consequence: environment reproduction from the provided "pinned versions" instruction (`README.md:49-55`) is incomplete.

8. The KL direction in the implementation is ambiguous relative to the paper theorem.
   - The paper defines original `p` and compressed `q` in `appendix.tex:29-32` and states `KL(p || q)` in `experiments.tex:203-210`.
   - In `transition_metrics_logging.py:100-105`, `q` is the residual/dense head and `p` is the output/pruned head, then `F.kl_div(log_q, p)` computes KL(pruned || dense). The README also says `KL(p_pruned || p_dense)` (`README.md:131`), so the code and README are aligned but the paper theorem notation is reversed.
   - Decision consequence: the plotted "ground-truth KL" is not clearly tied to the theorem without further clarification; because KL is asymmetric, this matters.

## Reproducibility Blockers

- No end-to-end script regenerates the paper figures or tables from raw benchmark outputs.
- No raw numeric results, seeds per benchmark, exact drop lists, exact checkpoint configs, generated-output logs, or plotting scripts are included.
- Qwen dropped-model support is incomplete/missing despite Qwen being the main paper model.
- Several scripts require manual path edits or external artifacts: README paths use `/path/to/...` (`README.md:110-123`, `README.md:203-216`, `README.md:247-259`); speed and quantization scripts use placeholder checkpoint paths (`inter-layer/scripts/benchmark/benchmark_speed.sh:3-10`, `inter-layer/scripts/quantization/awq.sh:3-4`, `inter-layer/scripts/quantization/gptq.sh:3-4`).
- The benchmark script downloads and mutates a local Mistral model in-place (`benchmark_lm_eval.sh:16-27`) and assumes directories such as `./mistral_drop8_attn/config.json` that are not produced or shipped by that script.

## Final Synthesis and Score Impact

Artifact completeness is weak. The repository contains useful fragments for pruning and representation logging, but it does not provide a reliable reproduction package for the paper's core empirical claims. The most serious issues are the missing benchmark reproduction paths, incomplete Qwen dropped-model support, likely no-op dropped analysis under the README commands, and a pruned attention hook that does not preserve autoregressive context. These are decision-relevant because the paper's acceptance case depends on measured discrepancies across pruning methods, models, and representation spaces. I would materially downgrade reproducibility confidence unless the authors provide exact configs/checkpoints/drop lists/raw results and corrected Qwen-compatible scripts.
