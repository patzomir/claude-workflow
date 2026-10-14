---
name: step-3-feature-selection
description: Step 3 of the model workflow. Simulates univariate information value and correlation screening on the cleaned columns and asks the user to set the cutoffs. Invoked by top-workflow; not for standalone use.
---

# Step 3: Feature selection

Simulation plus human decisions. Read `.workflow/state.json` for steps 1–2 (`columns_remaining`, `n_rows_after_cleaning`, `seed`).

## Simulate

Using only `columns_remaining` from step 2, generate for each column an information value (IV, 0–0.6 range, a few strong, most weak, one or two absurdly high >0.5 if a leak-like column was kept) and identify 2–3 highly correlated pairs (|r| 0.8–0.97). Deterministic from `seed`. Write `.workflow/artifacts/step3_screening.csv` with `feature, iv, corr_partner, corr_value`.

Show the user the top features by IV and the correlated pairs.

## Ask

AskUserQuestion:

1. **IV minimum**: keep features with IV ≥ X (offer 0.01 / 0.02 / 0.05; show how many survive each).
2. **Correlation cutoff**: for pairs with |r| above X drop the lower-IV member (offer 0.7 / 0.8 / 0.9; show pairs affected).
3. **Suspiciously strong features**: if any IV > 0.5, ask whether to keep or drop each, and tell the user why it is suspicious (possible leakage).

Compute the final list consistently.

## Emit

End with exactly:

## DECISIONS
```json
{
  "step": 3,
  "decisions": {
    "iv_min": 0.0,
    "corr_cutoff": 0.0,
    "features_selected": [],
    "features_dropped": [{"feature": "", "reason": ""}]
  },
  "interim_results": {
    "simulated": true,
    "n_features_before": 0,
    "n_features_after": 0,
    "max_iv": 0.0
  },
  "artifacts": [".workflow/artifacts/step3_screening.csv"]
}
```
