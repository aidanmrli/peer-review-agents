# Independent Reproducer B Report

Paper ID: a14d3e5d-d2c7-4877-bd15-45ee26effb81
Title: Whole-Brain Connectomic Graph Model Enables Whole-Body Locomotion Control in Fruit Fly
Assigned role: Independent Reproducer B
Date: 2026-04-26

## Task Scope

Use an independent route from Reproducer A: validate the central claim through table arithmetic, baseline logic, and internal consistency rather than attempting a full training run.

## Evidence Examined

- Table 1 source lines `main.tex:226-249`.
- Baseline description lines `main.tex:254-258`.
- Training/evaluation claims lines `main.tex:193-224` and appendix lines `main.tex:422-456`.
- Project page rendered text and extracted links.

## Commands and Calculations

```bash
nl -ba papers/a14d3e5d-d2c7-4877-bd15-45ee26effb81/artifacts/source/main.tex | sed -n '226,248p'
awk 'BEGIN {
  printf "high_yaw_angle_reduction=(13.55-8.29)/13.55=%.4f (%.1f%%)\n", (13.55-8.29)/13.55, 100*(13.55-8.29)/13.55;
  printf "speed3_yaw0_pos_reduction=(0.0385-0.0364)/0.0385=%.4f (%.1f%%)\n", (0.0385-0.0364)/0.0385, 100*(0.0385-0.0364)/0.0385;
  printf "speed3_yaw7_pos_reduction=(0.0370-0.0364)/0.0370=%.4f (%.1f%%)\n", (0.0370-0.0364)/0.0370, 100*(0.0370-0.0364)/0.0370;
}'
```

Output:

```text
high_yaw_angle_reduction=(13.55-8.29)/13.55=0.3882 (38.8%)
speed3_yaw0_pos_reduction=(0.0385-0.0364)/0.0385=0.0545 (5.5%)
speed3_yaw7_pos_reduction=(0.0370-0.0364)/0.0370=0.0162 (1.6%)
```

## Observed Result

The Table 1 angle-error advantage over degree-preserving rewiring is numerically meaningful, especially at speed=3, yaw=7. However:

- The MLP baseline is described in `main.tex:254-258`, but Table 1 reports only Connectome, Degree-Preserving Rewiring Graph, and Erdos-Renyi Random Graph. The paper's abstract and introduction claim comparison against MLP, but the main quantitative table does not show it.
- Position-error superiority is not broad: the rewired graph is better at speed=2,yaw=0 and speed=3,yaw=4, while differences at speed=3,yaw=7 are only 1.6% relative to the rewired graph.
- The shaded Figure 3 training curves are not accompanied by numeric values, raw logs, or seed counts. The caption says "multiple training runs" but the paper does not state how many.
- There is an internal method ambiguity: `main.tex:118-122` defines signed synaptic-count weights, while `main.tex:376` says the connectome was simplified to an unweighted directed graph. The project page also describes an unweighted directed graph.

## Reimplementation Feasibility From This Route

Blocked. Even the table-level check cannot be traced to metric code or raw logs. The implementation path needed to decide whether angle error was computed consistently is absent.

## Agreement With Reproducer A

Agreement after both passes: both routes fail to reproduce the central result. Reproducer A fails at implementation availability and underspecified pipeline; this route independently finds that only small arithmetic claims can be checked and that core baseline/statistical support is incomplete.

## Match Status

Partial match only for simple table arithmetic; blocked for actual reproduction.

Confidence level: high for table omissions and arithmetic; moderate for statistical interpretation because seed-level data are not released.
