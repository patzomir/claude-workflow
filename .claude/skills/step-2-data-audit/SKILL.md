---
name: step-2-data-audit
description: Step 2 of the model workflow. Simulates a data profile (missingness, outliers, class balance) and asks the user to choose cleaning cutoffs. Invoked by top-workflow; not for standalone use.
---

# Step 2: Data audit

Simulation plus human decisions. Read `.workflow/state.json` for step 1 decisions and `seed`.

## Simulate

Invent a plausible profile for the problem described in step 1. Be deterministic: derive numbers from `seed` and the step 1 target text so a re-run gives the same profile. Mark everything as simulated.

Produce and write to `.workflow/artifacts/step2_profile.csv` one row per column (12–20 columns, realistic names for the domain) with: `column, dtype, pct_missing, n_unique, min, max, pct_outliers_iqr`. Make the data interesting: a few columns with >40% missing, one with ~95% missing, one near-constant, one suspicious leak-like column (e.g. a field populated after the event), and an observed positive rate that is near but not identical to step 1's expectation. Total rows: 20k–200k.

Show the user a compact table of the 8 most notable columns plus the headline numbers (`n_rows`, `observed_positive_rate`).

## Ask

With AskUserQuestion, get:

1. **Missing threshold**: drop columns with more than X% missing (offer 30/50/70%, show how many columns each would drop).
2. **Outlier rule**: none / winsorize at 1–99 / winsorize at 5–95 / drop rows beyond 3×IQR (show rows lost).
3. **Columns to drop manually**: multi-select from the notable list (the leak-like column is a trap; mention the suspicion neutrally, don't decide for the user).

Compute `n_rows_after_cleaning` consistently with the choices (simulated but arithmetic must add up).

## Emit

End with exactly:

## DECISIONS
```json
{
  "step": 2,
  "decisions": {
    "missing_threshold": 0.0,
    "outlier_rule": "",
    "columns_dropped": []
  },
  "interim_results": {
    "simulated": true,
    "n_rows": 0,
    "n_rows_after_cleaning": 0,
    "observed_positive_rate": 0.0,
    "columns_remaining": []
  },
  "artifacts": [".workflow/artifacts/step2_profile.csv"]
}
```
