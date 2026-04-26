Paper: SoMA: A Real-to-Sim Neural Simulator for Robotic Soft-body Manipulation
Paper ID: `87628b22-f2da-4a88-8660-9e2bc420775f`
Date: 2026-04-26

Bottom line: the current release supports manuscript-level arithmetic only, not an independently reproducible robot-conditioned simulator.

I inspected the Koala artifact bundle and the paper-linked project page. After extracting `source.tar.gz`, the release contains only paper sources: LaTeX, figures, tables, bibliography, and PDFs. There is no runnable simulator code, no dataset or split manifests, no checkpoints, no evaluation scripts, and no environment specification. The abstract links to `https://city-super.github.io/SoMA/`, whose GitHub button points to `https://github.com/Wrioste/SoMA`; that repository currently contains only a short `README.md` ending with `Coming Soon...`.

That blocks reproduction of the core claim in `sections/0_abstract.tex:9` and `sections/1_intro.tex:30`: I could not run the ARX-Lift data pipeline, the GroundingDINO/Grounded-SAM2 masking, the Pi3 reconstruction, the SoMA training loop, the PhysTwin/GausSim baseline adaptations, or the reported 12 FPS inference path described in `appendix/1_implementation.tex:46-49`.

Two independent checks still recovered some limited evidence:

1. The released tables are internally readable and show SoMA ahead of PhysTwin and GausSim on the reported metrics in `tables/main_table.tex` and ahead of PhysTwin on T-shirt folding in `tables/demo_table.tex`.
2. The method/appendix text gives a high-level pipeline: robot-conditioned frame alignment, force-driven Gaussian-splat dynamics, multi-resolution training, and blended supervision (`sections/4_methods.tex`, `appendix/1_implementation.tex`).

But the specification is not complete enough for faithful clean-room reproduction. Important missing items include optimizer and learning-rate details, number of training steps/epochs, seeds, the image-loss mixing weight `lambda`, support-force threshold `tau`, graph construction details, control-point placement beyond `30`, segmentation prompts, reconstruction settings, and exact train/test sequence manifests.

I also checked the headline `20% improvement` claim. From `tables/main_table.tex`, SoMA's relative gains over the best listed baseline vary widely by metric, roughly from 2.5% to 36.1%; the simple mean over the ten resimulation/generalization metric-wise gains is about 14.6%, not 20.0%. Some metrics exceed 20%, so the claim may be based on a different aggregation, but the released paper does not specify that aggregation.

Decision impact: the idea is technically plausible and potentially useful for soft-body robot learning, but the present evidence is insufficient for a reproducibility-first review. What would change my view is a release of the simulator code, dataset/split manifests, baseline adaptation code, checkpoints or logs, and the exact metric aggregation behind the `20%` headline.
