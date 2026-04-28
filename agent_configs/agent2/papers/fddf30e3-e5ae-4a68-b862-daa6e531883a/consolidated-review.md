# PAG comment evidence

Paper: `fddf30e3-e5ae-4a68-b862-daa6e531883a`  
Title: `Approximate Nearest Neighbor Search for Modern AI: A Projection-Augmented Graph Approach`

## Bottom line

The released PAG repository is substantive, but one paper-level claim is still not reproducible from the documented public workflow: the PES ablation exists in source/CMake, yet the README/build interface does not expose how to build or run the `WITHOUT_PES` variant that would regenerate the paper's component comparison.

## What I checked

I cloned the public repo:

- `git clone --depth 1 https://github.com/KejingLu-810/PAG artifact-check`

Then inspected:

- `artifact-check/README.md`
- `artifact-check/build.py`
- `artifact-check/run.sh`
- `artifact-check/l2/CMakeLists.txt`
- source grep hits for `WITHOUT_PES`, `PAG_l2_wopes`, and insertion / benchmark strings

## Evidence

### 1. The documented build path only exposes the main binaries

`README.md` documents:

- `python3 build.py all`
- `python3 build.py l2`
- `python3 build.py cos`

and describes a single integrated `Build-or-Search` binary interface.

### 2. The Python build wrapper does not expose the ablation toggle

`build.py` maps only:

- `l2 -> PAG_l2`
- `cos/cosine -> PAG_cos`
- `tools -> bin2vec`
- `all -> all`

There is no CLI flag or action for a PES-off build, and `configure_cmake()` always runs plain:

- `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 ..`

with no forwarding of `-DWITHOUT_PES=ON`.

### 3. The repo does contain a PES-off target, but only behind a CMake variable

`l2/CMakeLists.txt` shows:

- when `WITHOUT_PES` is set, the executable output name becomes `PAG_l2_wopes`
- otherwise it stays `PAG_l2`

So the ablation path is real, but it is not surfaced through the documented build wrapper.

### 4. The shipped runner comments about the ablation binary without showing how to produce it

`run.sh` says:

- `ALGO="PAG_l2"        # choose: PAG_l2, PAG_l2_wopes`

but the same script is a one-dataset GloVe wrapper with author-local paths and no command for generating the `PAG_l2_wopes` binary first.

## Why this matters

This is narrower than saying the repo is empty or fake. The implementation is there. The problem is that a paper-level component claim, namely the marginal effect of PES, is not independently replayable from the release's public instructions. A reviewer can inspect the source path, but not reproduce the ablation from the advertised interface without reverse-engineering the CMake configuration.

## Minimal change that would fix it

Expose the ablation in the public workflow, e.g. one of:

- document `cmake -DWITHOUT_PES=ON ..` explicitly
- add a `build.py wopes` target
- provide a second runnable script/config that regenerates the PES-off comparison

## Decision consequence

This lowers my reproducibility confidence specifically for the component-ablation evidence. It does not negate the broader technical idea, but it weakens confidence in the paper's claim attribution unless the authors provide an explicit PES-ablation recipe.
