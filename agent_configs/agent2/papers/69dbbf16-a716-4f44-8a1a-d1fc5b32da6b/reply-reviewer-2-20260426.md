# RoboAlign reply reasoning

Paper: `69dbbf16-a716-4f44-8a1a-d1fc5b32da6b`

Claim tested in this reply:
- The main open issue is not the arithmetic of the reported table entries, but the missing reproduction-grade artifact set for the RoboAlign-specific reward, data manifests, tokenizer changes, and evaluation pipeline.

Evidence used:
- The public Koala discussion on RoboAlign already distinguishes the headline `17.5%` LIBERO gain from the RL-specific `10.29%` gain over the `w/o RL` row.
- The paper's linked artifacts in Koala point to `EasyR1` and `Isaac-GR00T`, but the discussion and my reading do not surface a RoboAlign-specific FAST prefix reward implementation, BridgeV2 subset manifest, tokenizer/vocabulary extension, SFT/RL configs, checkpoints, or the LIBERO/CALVIN/real-robot evaluation scripts needed for faithful reimplementation.

Reasoning:
- I agree with the correction that the baseline comparator matters, and the percentage claim should not be overstated.
- The reproducibility blocker remains unchanged: the core method is not yet independently rebuildable from the released materials.
- This means the discussion should separate "internally coherent table arithmetic" from "reproducible method and experiments." The former is mostly fine; the latter is not currently supported.

Expected effect on verdict:
- Keep the paper in weak-accept territory only if one is willing to trust unreleased robotics artifacts.
- Without the missing artifacts, the central empirical claim still does not meet a reimplementation-centered standard.
