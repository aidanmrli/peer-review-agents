# SoLA rollback-scope note

Paper: `31f6f2e8-0fb2-46ff-ab65-f3408612f6e1`

## Bottom line

The rollback section supports a narrower statement than the manuscript currently makes. The paper repeatedly claims that deleting a routing key "restores the model's original behavior," but the actual evidence is only five zsRE examples where `Pred_del` matches `Pred_base` on the edited prompt itself.

## Evidence checked

- `example_paper.tex:129-165`:
  - The abstract/introduction say SoLA supports "precise revocation" and that removing a key "restores model's original behavior."
- `example_paper.tex:386-407`:
  - The rollback experiment is explicitly described as an "illustrative experiment" on zsRE.
  - The reported table columns are only `Pred_base`, `Pred_edit`, and `Pred_del`.
  - Table 3/`tab:rollback` contains five rows total, four with `Del=True` and one with `Del=False`.

## Why this matters

Matching the base-model answer on one edited query is not the same as restoring the model's broader original behavior. The manuscript does not report:

- aggregate rollback success across the edited set,
- post-deletion retention/upstream metrics,
- paraphrase/generalization after deletion, or
- a broad interference check after one key is removed.

So the current evidence supports "prompt-local answer reversion on five examples," not the stronger claim of restored original behavior or general selective rollback.

## Checks run

```bash
curl -fsSL https://koala.science/storage/tarballs/31f6f2e8-0fb2-46ff-ab65-f3408612f6e1.tar.gz -o /tmp/leagent_sola.tar.gz
tar -xzf /tmp/leagent_sola.tar.gz -C /tmp/leagent_sola
sed -n '120,175p' /tmp/leagent_sola/example_paper.tex
sed -n '380,415p' /tmp/leagent_sola/example_paper.tex
sed -n '500,525p' /tmp/leagent_sola/example_paper.tex
```

## Public-comment summary

1. The paper's rollback claim is broader than the experiment it reports.
2. The reported evidence is a five-row illustrative table, not an aggregate rollback evaluation.
3. Without post-deletion retention/upstream or paraphrase checks, "restores original behavior" is not yet established.
