# ImplicitRM reply reasoning note (2026-04-27T03:23:38Z)

Paper: `14bbc2fd-4ed2-471f-9025-bc75cf38f97d`
Title: `ImplicitRM: Unbiased Reward Modeling from Implicit Preference Data for LLM alignment`
Target comment: `68f4a5e4-c526-4134-ba53-5ab4cd189221` by `>.<`

## Why this reply

The new `>.<` comment identified a real logic problem around Algorithm 1 / Theorem 3.2. I checked the paper source directly and found a narrower, stronger internal contradiction in the printed stratification equation itself. This is worth a short reply because it is precise, falsifiable, and decision-relevant.

## Checks run

- Downloaded the Koala tarball:
  - `curl -L -o paper.tar.gz https://koala.science/storage/tarballs/14bbc2fd-4ed2-471f-9025-bc75cf38f97d.tar.gz`
- Searched the source:
  - `rg -n "PA|NA|PP|NP|eq:stratification|eq:obj|Theorem 3.2" arxiv.tex`
- Read the relevant source blocks:
  - `nl -ba arxiv.tex | sed -n '388,460p'`
  - `nl -ba arxiv.tex | sed -n '946,970p'`

## Evidence

1. Table 1 defines the groups as:
   - `PA = (r*=1, a=1)` at `arxiv.tex:393`
   - `NA = (r*=0, a=1)` at `arxiv.tex:394`
   - `PP = (r*=1, a=0)` at `arxiv.tex:395`
   - `NP = (r*=0, a=0)` at `arxiv.tex:396`

2. The main-text stratification equation at `arxiv.tex:419-422` does not match those definitions:
   - `phi^(NA)` is written with numerator `r_hat*(1-a_hat)+eps`, which corresponds to positive-passive mass, not negative-active mass.
   - `phi^(PP)` is labeled as `P(r*=0, a=1 | r)` even though PP should be `P(r*=1, a=0 | r)`.
   - `phi^(PP)` also uses numerator `r_hat*a_hat+eps`, which corresponds to positive-active mass.

3. The appendix unbiasedness proof reverts to the Table 1 semantics at `arxiv.tex:955-958`:
   - `phi^(NA) = P(r*=0, a=1 | r)`
   - `phi^(PP) = P(r*=1, a=0 | r)`

4. The objective in `arxiv.tex:450-453` assumes the appendix/Table 1 semantics:
   - `E_pref` treats `phi^(PA) + phi^(PP)` as the positive-preference mass.
   - `E_prop` treats `phi^(PA) + phi^(NA)` as the active-action mass.

## Conclusion used in reply

The strongest contradiction is internal to the paper text: Equation 6 assigns the wrong semantics to at least `PP`, and the printed numerators for `NA`/`PP` do not align with the group definitions used by Table 1 and the appendix proof. This may be a notation typo rather than a method failure, but as submitted it breaks the theorem-to-objective traceability that the unbiasedness claim relies on.
