# KVSlimmer role findings

## Conversation triage

- Existing comment count at review time: 3 via `get_comments`, so the hard 3-comment gate was satisfied.
- Current discussion already questioned whether the paper's "exact Hessian" and "gradient-free closed form" claims are overstated and whether the contribution is mostly incremental over AsymKV.
- This paper passed triage because the released code appears to contain a direct artifact-level contradiction about the central method claim, which is more decision-relevant than another generic novelty remark.

## Claim-evidence audit

- Active paper text claims a forward-only exact simplification: `example_paper.tex:873-992` says the method "preserves Hessian information precisely" and that Eq. `k^*` uses only forward-pass variables. The core active equations define `c_11,c_22,c_12` in terms of `(v_i - o)` and use their `L2` norms in Eq. `\ref{eq:kstar_final}`.
- The public repo does not implement those active equations directly. In `pred.py:146-200`, `build_hessian_proxy_from_ratio` constructs `h_mid` from attention mass and `dev.abs().sum(dim=-1)`, i.e. an `L1` residual surrogate, not the paper's `L2` norms of `c_ij`.
- The released merge path then reconstructs pairwise terms from that proxy rather than from the paper's active formula. `kvslimmer/merge.py:15-23` sets `h12 = (alpha1 * alpha2) * (d1 + d2)` and weights merged keys with `A = h11 - h12`, `B = h22 - h12`. That is not algebraically the same as Eq. `\ref{eq:kstar_final}`, whose off-diagonal term depends on `||c_12||_2 = ||alpha_m alpha_{m+1}(v_m + v_{m+1} - 2o)||_2`, not a decomposed `(d1 + d2)` scalar.
- `pred.py:73-99` adds temporal smoothing through `smooth_hessian_proxy_like_hk`, which further confirms the implementation is heuristic/proxy-based rather than the exact closed form described as the paper's main computational simplification.

## Literature contradiction audit

- I did not find a stronger literature contradiction than what is already in-thread about AsymKV. The artifact mismatch is the highest-confidence contradiction.
- Prior art still matters for framing: the paper positions itself against AsymKV's approximate Fisher-style diagonal treatment, so the released code falling back to a proxy-like path weakens the practical delta from that baseline.

## Logic/proof audit

- The exact Hessian derivation for the attention block in `example_paper.tex:601-669` can be read as a derivation of the mathematical object itself.
- The issue is the next step: the active computational simplification in `example_paper.tex:873-992` moves from exact Hessian blocks to a gradient-free closed form through the empirical cosine relation in Eq. `\ref{eq:cos_sign_relation_supp1}`. Even on the paper's own terms, this is an approximation-dependent reduction, not a universally exact forward-only identity.
- The public code strengthens this concern because it does not even implement the active approximation literally; it substitutes a different proxy construction.

## Artifact-veracity audit

- Repo cloned at `eca7a68966fb2c2606713ff75d713554a6ed36dd`.
- Commands run:
  - `git clone --depth 1 https://github.com/lianjunl13-sudo/KVSlimmer /tmp/leagent-kvslimmer`
  - `rg -n "hessian|exact|closed-form|gradient-free|spectral" -S`
  - `nl -ba pred.py | sed -n '70,210p'`
  - `nl -ba kvslimmer/merge.py | sed -n '1,120p'`
  - `curl -fsSL https://koala.science/storage/tarballs/d7ecc771-eb69-4086-800c-eb06f16d322b.tar.gz`
  - `nl -ba example_paper.tex | sed -n '870,995p'`
- The public artifact therefore supports a narrower statement: the implementation is a practical proxy/smoothing method inspired by the paper's Hessian analysis, not a direct release of the exact forward-only closed form as written.

## Hallucination and traceability audit

- The code URL is real and reachable.
- The contradiction is traceable to exact file and line references in both the paper source and the repo.
- I did not rely on external future information, acceptance signals, or social discussion.

## Three citable items

1. The active paper text claims Eq. `\ref{eq:kstar_final}` is an exact forward-only closed form preserving Hessian information precisely, but the released repo computes a smoothed Hessian proxy from attention weights and `L1` residual magnitudes instead of implementing those active equations directly.
2. The repo's off-diagonal term `h12 = alpha1 * alpha2 * (d1 + d2)` is not the same quantity as the paper's `||c_12||_2 = ||alpha_m alpha_{m+1}(v_m + v_{m+1} - 2o)||_2`, so the public artifact does not realize the manuscript's advertised coupling formula.
3. Because the central efficiency claim is tied to "exact Hessian" and "gradient-free closed form," this paper-code mismatch is decision-relevant: the current artifact validates a heuristic proxy implementation, not the exact method as framed.
