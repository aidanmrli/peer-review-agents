# Sign Lock-In: consolidated review evidence

Paper: `0ce14447-2762-4440-9dcc-e65edac3e7e5`  
Title: `Sign Lock-In: Randomly Initialized Weight Signs Persist and Bottleneck Sub-Bit Model Compression`

## Bottom line

The sign-lock-in phenomenon itself looks real and decision-relevant, but the paper's strongest practical compression result is narrower than the headline framing suggests: in the appendix, the reported sub-bit setup explicitly keeps non-targeted parameters in full precision and enforces exact sign templates with post-update hard projection.

## What I checked

Commands / actions run in this cycle:

```bash
curl -L --fail --silent https://koala.science/storage/tarballs/0ce14447-2762-4440-9dcc-e65edac3e7e5.tar.gz -o tmp/sign-lockin.tar.gz
tar -xzf tmp/sign-lockin.tar.gz -C tmp/sign-lockin-src
rg -n 'template|hard projection|all other parameters are maintained in full precision|bpw|sub-bit|one-bit wall' tmp/sign-lockin-src/main.tex
sed -n '760,790p' tmp/sign-lockin-src/main.tex
sed -n '4410,4478p' tmp/sign-lockin-src/main.tex
sed -n '4575,4705p' tmp/sign-lockin-src/main.tex
```

## Evidence recovered

### 1. The main-text intervention and the strongest appendix result are not the same thing

- In the main text, the paper presents gap initialization and outward-drift regularization as the minimal interventions that suppress sign flips and keep perplexity roughly intact.
- The same section then explicitly says the strongest form is deferred to the appendix: exact template enforcement that makes sign storage effectively zero.

This matters because the paper's practical compression takeaway leans on the appendix result, not just the main-text passive lock-in evidence.

### 2. The appendix compression result uses exact hard projection after every optimizer step

- The appendix states:
  - "After each optimizer update, we perform an element-wise hard projection"
  - `Pi_hard(W)_{ij} = T_{ij} * |W_{ij}|`
- It further says this enforces `sign(W)=T` exactly for all targeted layers and "guarantees" zero sign storage cost at compression time.

So the practical compression result is not simply "signs naturally stay put." It is constrained training with exact sign correction.

### 3. The reported sub-bit experiment is selected-layer only, not whole-model

- The appendix says the zero-template method is applied to a fixed set of targeted weight matrices (linear layers), while "all other parameters are maintained in full precision."
- The listed targeted tensors are:
  - CharLM / Text8-Char: 14 tensors
  - DBPedia14: 28 tensors

That makes the experiment useful as a targeted-layer compression proof-of-concept, but it is weaker than a whole-model sub-bit deployment claim.

### 4. Bit accounting is explicit for targeted matrices, but the headline framing can be overread

- The appendix gives `bpw_eff` accounting for the compressed matrices and correctly contrasts sign-free template storage with SVD-on-raw-weights and sparse CSR baselines.
- However, because untouched parameters remain full precision, Figure G.6 should be read as a targeted-layer storage comparison unless the authors explicitly report end-to-end whole-model memory.

## Two-pass conclusion

- Positive: the paper identifies a real sign-persistence phenomenon and provides a concrete way to make targeted-layer sign storage effectively zero.
- Negative: the practical compression headline currently bundles together three different things:
  - observing natural sign lock-in,
  - encouraging it with gap/regularization,
  - and enforcing it exactly with post-update hard projection on selected layers.

## Public-comment takeaway

The useful public point is not that the paper lacks technical substance. It is that the appendix's strongest compression result should be framed as selected-layer constrained training with exact sign-template enforcement, not as a pure consequence of natural sign persistence or as a whole-model sub-bit deployment result.
