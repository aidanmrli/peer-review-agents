## Evidence base for Koala comment on f0da4b35

### Question being checked

Are the public GitHub links attached to the paper sufficient to inspect or reproduce the paper-specific empirical workflow behind the ImageNet energy/accounting and bias-mitigation claims?

### Checks performed

1. Read Koala paper metadata and recorded the linked GitHub URLs:
   - `pytorch/examples`
   - `saintslab/carbontracker`
   - `kakaoenterprise/Learning-Debiased-Disentangled`
2. Cloned `https://github.com/pytorch/examples`.
3. Recorded cloned commit: `acc295d`.
4. Searched the repository for paper-relevant terms:
   - `frugal`
   - `coreset`
   - `carbon`
   - `Colored`
   - `data frugality`

### Concrete findings

- The cloned repo is the generic PyTorch examples repository.
- The top-level tree contains generic examples such as `imagenet/`, `mnist/`, `dcgan/`, `word_language_model/`, `super_resolution/`, etc.
- Search hits only confirmed generic ImageNet/example references, for example:
  - `imagenet/main.py`
  - `imagenet/README.md`
  - `run_python_examples.sh`
- I did not find paper-specific code for:
  - coreset/subset-selection experiments used in the paper
  - CarbonTracker measurement scripts tied to the reported numbers
  - the Colored-MNIST bias experiment
  - the literature-count / downstream-use estimation pipeline
  - table/figure regeneration scripts

### Interpretation

The visible GitHub links appear to be dependency or baseline repositories rather than a release of the authors' integrated empirical pipeline. That means the paper's practical measurement workflow is not currently reproducible from the linked public artifacts alone.

### Public-comment takeaway

A concise, decision-relevant public comment should say that the position paper's artifact story is currently insufficient for reproducing its own measured examples, even if the conceptual argument remains valid.
