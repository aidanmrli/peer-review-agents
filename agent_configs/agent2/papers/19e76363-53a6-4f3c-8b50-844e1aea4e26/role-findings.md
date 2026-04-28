# Med-TIV Reproducibility Findings

## Central claim and reproduction target

The paper claims Med-TIV is a tool-integrated medical reasoning verifier that improves benchmark accuracy and achieves an 8x sampling-budget reduction relative to prior reward-model baselines. My reproduction target was the released artifact path for the tool-integrated inference stack, since that stack is load-bearing for both the retrieval-grounding claim and the sampling-efficiency claim.

## Paper and artifact evidence checked

- Paper source tarball at `19e76363-53a6-4f3c-8b50-844e1aea4e26.tar.gz`, especially `example_paper.tex`:
  - abstract / intro claims on dynamic retrieval and 8x efficiency
  - Section 2.2 iterative tool-use setup
  - Section 4.3 sampling-budget claim
  - Appendix retrieval setup
- Public repo `https://github.com/PittNAIL/med-tiv` at commit `f588c39`
- Repo files inspected:
  - `README.md`
  - `inference/run_medical_judge_inference_multi_file.sh`
  - `examples/train/search_r1/train_7b_prm_ncbi.sh`

## Smallest meaningful check actually run

I performed an artifact integrity check on the release instructions:

- cloned the repo and enumerated files
- verified whether the README quickstart entrypoints exist
- inspected the inference wrapper for hard-coded runtime dependencies

Concrete result:

- `README.md` tells users to run `python inference/medical_dense_retrieval_tool.py` in the manual inference path
- `inference/run_medical_judge_inference_multi_file.sh` also kills and launches `medical_dense_retrieval_tool.py`
- that file is absent from the released repo at commit `f588c39`

This is a stricter blocker than merely needing local path filling: the documented tool-integrated inference route cannot be executed from the release as written because one of its referenced entrypoints is missing.

## Implementation or correctness risks

- The main contribution is specifically tool-integrated verification; a missing tool-server entrypoint weakens reproducibility of the core mechanism, not just convenience scripts.
- The same wrapper script also requires unpublished local paths for the index, corpus, model checkpoint, and benchmark traces, so the release still lacks end-to-end experiment material even aside from the missing file.
- The training script leaves `model_name`, `train_data`, `val_data`, and `retriever_path` empty, which further limits exact replay.

## Novelty or framing context

This finding does not argue against the idea of iterative retrieval itself. It narrows the reproducibility claim: the release looks like a serious scaffold, but the public artifact does not yet support direct reproduction of the paper's tool-integrated inference pipeline.

## Decision impact

My check partially supports that there is substantive code, but it does not support the stronger claim that an external reviewer can independently rerun the reported Med-TIV verification stack from the public release. Releasing the missing tool entrypoint and experiment-ready inference assets would materially improve confidence.
