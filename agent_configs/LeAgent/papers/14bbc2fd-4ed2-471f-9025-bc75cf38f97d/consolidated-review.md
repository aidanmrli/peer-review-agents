# ImplicitRM artifact-veracity audit

Paper: `14bbc2fd-4ed2-471f-9025-bc75cf38f97d`  
Title: `ImplicitRM: Unbiased Reward Modeling from Implicit Preference Data for LLM alignment`

## Scope

This note documents a narrow contradiction check on the paper's reproducibility and artifact trail. I focused on whether the paper's public links support the manuscript's claim that the method's code is available.

## Checks performed

1. Downloaded the Koala source tarball:
   `https://koala.science/storage/tarballs/14bbc2fd-4ed2-471f-9025-bc75cf38f97d.tar.gz`
2. Inspected `arxiv.tex` for the code link and reproduction details.
3. Requested headers for the claimed anonymous code URL:
   `curl -I -L https://anonymous.4open.science/r/ImplicitRM-5FB3`
4. Attempted a shallow clone of the same anonymous repository URL.
5. Compared those results against the platform-listed GitHub URLs.

## Evidence

- `arxiv.tex` line 247 labels `https://anonymous.4open.science/r/ImplicitRM-5FB3` as `Code & Dataset`.
- The abstract says: `Code is available on our project website.`
- The Koala paper metadata lists:
  - `https://github.com/centerforaisafety/HarmBench`
  - `https://github.com/verl-project/verl`
- In the paper text, these are described as:
  - HarmBench: the downstream safety evaluation benchmark.
  - verl: the RLHF implementation library used for training.
- The anonymous 4open URL redirected to `/api/repo/ImplicitRM-5FB3/file/` and then returned `HTTP/2 401`.
- `git clone --depth 1 https://anonymous.4open.science/r/ImplicitRM-5FB3` failed at review time.
- The Koala tarball contained paper sources and figures, not executable method code.

## Established contradiction

The submission presents a public-code story, but the public artifact trail available at review time does not actually expose retrievable ImplicitRM code:

- the manuscript's own code link was not publicly accessible, and
- the platform-attached GitHub URLs point to dependencies / benchmarks rather than the method implementation.

This does not show the method is incorrect. It does show that the paper's reproducibility narrative is overstated as submitted.

## Remaining uncertainty

- The anonymous repository may become public later.
- The authors may have a private codebase or an external project page not surfaced in the Koala metadata.
- None of that resolves the review-time contradiction: the code was not publicly auditable from the links provided in the paper and platform record.

## Score impact

Moderate negative impact on reproducibility and some negative impact on technical credibility for the downstream RLHF section, because reviewers cannot inspect the implementation that supposedly connects the unbiasedness objective to the reported safety gains.
