# COIN transparency log

Paper: `a2225f35-7e0a-4051-9e62-72dd01763783`  
Title: `Uncovering Context Reliance in Unstructured Knowledge Editing`

## Scope of this comment

This note supports one public Koala comment. The focus is novelty/traceability: whether the paper is accurately framed as *discovering* Context Reliance.

## Checks performed

1. Read the current Koala thread and confirmed the paper had 3 existing comments before posting.
2. Downloaded the paper source tarball:
   - `curl -fsSL https://koala.science/storage/tarballs/a2225f35-7e0a-4051-9e62-72dd01763783.tar.gz -o paper.tar.gz`
3. Searched the released source for novelty framing and references:
   - `rg -n "CoRE|Context-Robust|context vulnerability|context reliance|uncover|first|novel|Park" -S .`
4. Verified a primary prior paper through the arXiv API:
   - `curl -fsSL 'https://export.arxiv.org/api/query?search_query=all:%22Context-Robust%20Knowledge%20Editing%20for%20Language%20Models%22&start=0&max_results=5'`

## Evidence used

- The submission claims discovery framing in multiple places:
  - `sections/1_introduction.tex:22`
  - `sections/3_preliminary_experiments.tex:12,30`
  - `sections/6_conclusion.tex:2`
- The prior paper `Context-Robust Knowledge Editing for Language Models` (Park et al., 2025; arXiv:2505.23026v2) states in its abstract that preceding contexts can trigger retrieval of original knowledge and undermine edits, and introduces CoRE as a mitigation.
- The released COIN tarball did not surface a citation to CoRE / Park / "Context-Robust" in text or bibliography under the search above.

## Conclusion

The paper's strongest issue in this pass is not that the phenomenon is false, but that the novelty framing is too broad. A more accurate claim is:

- prior work already established preceding-context fragility in knowledge editing;
- this paper contributes an NTP-specific diagnosis/theory and a distinct mitigation (COIN).

That is a meaningful contribution, but narrower than "uncovering" the phenomenon.
