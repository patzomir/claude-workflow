---
name: step-4-model-training
description: Step 4 of the model workflow. Simulates cross-validated training of candidate model families and lets the user pick one. Invoked by top-workflow; not for standalone use.
---

# Step 4: Model training

Simulation plus human decision. Read `.workflow/state.json` for steps 1–3.

## Simulate

Simulate 3 candidates: regularised logistic regression, random forest, gradient boosting. For each, invent cross-validated results that respect the data: AUC ceiling should be sensible given the strongest IV from step 3 (max_iv ~0.3 → AUC around 0.70–0.80; do not exceed 0.85 unless a leak-like feature was kept, in which case inflate and let the user see it). Typical ordering: GBM ≥ RF ≥ LR on AUC, but with larger train/val gap for GBM/RF. Deterministic from `seed`.

Write `.workflow/artifacts/step4_candidates.csv` with `model, cv_folds, train_auc, val_auc, val_primary_metric, fit_seconds` where `val_primary_metric` is the step 1 primary metric. Show it as a table.

## Ask

AskUserQuestion:

1. **Model family**: one of the three (state the trade-off: interpretability vs. lift vs. overfit gap).
2. **CV folds**: 3 / 5 / 10 (affects the simulated stability; report fold std dev).
3. **Key hyperparameter** for the chosen family (e.g. C for LR, max_depth for RF/GBM) from 3 offered values, with a one-line effect on the gap.

Update the chosen model's simulated metrics to reflect the folds and hyperparameter choice (modest, plausible shifts) and report them.

## Emit

End with exactly:

## DECISIONS
```json
{
  "step": 4,
  "decisions": {
    "model_family": "",
    "hyperparameters": {},
    "cv_folds": 0
  },
  "interim_results": {
    "simulated": true,
    "train_auc": 0.0,
    "val_auc": 0.0,
    "val_auc_std": 0.0,
    "val_primary_metric": 0.0
  },
  "artifacts": [".workflow/artifacts/step4_candidates.csv"]
}
```
