---
name: step-5-cutoff-selection
description: Step 5 of the model workflow. Simulates the precision/recall/cost curve over a probability-cutoff grid and lets the user choose the operating cutoff. Invoked by top-workflow; not for standalone use.
---

# Step 5: Cutoff selection

Simulation plus human decision. Read `.workflow/state.json` for steps 1–4 (costs, positive rate, `val_auc`, `n_rows_after_cleaning`).

## Simulate

Build a cutoff grid 0.05–0.95 step 0.05. For each cutoff produce confusion counts on a validation set of 20% of `n_rows_after_cleaning`, using the observed positive rate and a score distribution consistent with the step 4 `val_auc`. Numbers must be arithmetically coherent: TP+FN = positives, TP+FP = predicted positives, precision = TP/(TP+FP), recall = TP/(TP+FN), expected_cost = cost_fp·FP + cost_fn·FN (from step 1). Deterministic from `seed`.

Write `.workflow/artifacts/step5_cutoff_grid.csv` with `cutoff, tp, fp, fn, tn, precision, recall, f1, expected_cost`. Show a compact table of every other row and mark: the cost-minimising cutoff, the max-F1 cutoff, and the cutoff meeting `min_acceptable_metric` if the metric is directly readable from the grid.

## Ask

AskUserQuestion:

1. **Selection rule**: minimise expected cost / maximise F1 / fixed recall target / fixed precision target (explain which is consistent with step 1's primary metric; the user may still deviate).
2. If a fixed target rule: the target value.
3. Confirm the resulting cutoff, showing precision, recall, flagged-volume and expected cost at it. Offer the neighbouring cutoffs as alternatives.

## Emit

End with exactly:

## DECISIONS
```json
{
  "step": 5,
  "decisions": {
    "selection_rule": "",
    "cutoff": 0.0,
    "grid": [0.05, 0.10, 0.15, 0.20, 0.25, 0.30, 0.35, 0.40, 0.45, 0.50, 0.55, 0.60, 0.65, 0.70, 0.75, 0.80, 0.85, 0.90, 0.95]
  },
  "interim_results": {
    "simulated": true,
    "tp": 0, "fp": 0, "fn": 0, "tn": 0,
    "precision": 0.0,
    "recall": 0.0,
    "expected_cost": 0.0
  },
  "artifacts": [".workflow/artifacts/step5_cutoff_grid.csv"]
}
```
