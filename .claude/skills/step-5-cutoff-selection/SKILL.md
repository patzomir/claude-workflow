---
name: step-5-cutoff-selection
description: Step 5 of the model workflow. Simulates the precision/recall/cost curve over a probability-cutoff grid and lets the user choose the operating cutoff. Invoked by top-workflow; not for standalone use.
---

# Step 5: Cutoff selection

Simulation plus human decision. Read `seed` from `.workflow/state.json` and the decision files `.workflow/decisions/step-1.md` to `step-4.md` (costs, positive rate, validation AUC, rows after cleaning).

## Simulate

Build a cutoff grid 0.05–0.95 step 0.05. For each cutoff produce confusion counts on a validation set of 20% of `n_rows_after_cleaning`, using the observed positive rate and a score distribution consistent with the step 4 `val_auc`. Numbers must be arithmetically coherent: TP+FN = positives, TP+FP = predicted positives, precision = TP/(TP+FP), recall = TP/(TP+FN), expected_cost = cost_fp·FP + cost_fn·FN (from step 1). Deterministic from `seed`.

Write `.workflow/artifacts/step5_cutoff_grid.csv` with `cutoff, tp, fp, fn, tn, precision, recall, f1, expected_cost`. Show a compact table of every other row and mark: the cost-minimising cutoff, the max-F1 cutoff, and the cutoff meeting `min_acceptable_metric` if the metric is directly readable from the grid.

## Ask

AskUserQuestion:

1. **Selection rule**: minimise expected cost / maximise F1 / fixed recall target / fixed precision target (explain which is consistent with step 1's primary metric; the user may still deviate).
2. If a fixed target rule: the target value.
3. Confirm the resulting cutoff, showing precision, recall, flagged-volume and expected cost at it. Offer the neighbouring cutoffs as alternatives.

## Emit

End your reply with a `## DECISIONS` heading followed by a free-text markdown record and nothing after it. Write it for a reader who has seen none of the conversation (a person or another LLM): plain prose or bullets, no required schema. Include selection rule, chosen cutoff, the confusion counts, precision, recall and expected cost at it (with the costs used), the neighbouring cutoffs considered, and a note that the grid is simulated. Give the reason behind each choice in the user's own words where you have them. State every number with its unit. List the artifact path: `.workflow/artifacts/step5_cutoff_grid.csv`.
