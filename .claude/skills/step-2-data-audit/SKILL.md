---
name: step-2-data-audit
description: Step 2 of the model workflow. Simulates a data profile (missingness, outliers, class balance) and asks the user to choose cleaning cutoffs. Invoked by top-workflow; not for standalone use.
---

# Step 2: Data audit

Simulation plus human decisions. Read `.workflow/state.json` for `seed` and `.workflow/decisions/step-1.md` for step 1 decisions.

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

End your reply with a `## DECISIONS` heading followed by a free-text markdown record and nothing after it. Write it for a reader who has seen none of the conversation (a person or another LLM): plain prose or bullets, no required schema. Include missing threshold, outlier rule, columns dropped (and why), rows before/after cleaning, observed positive rate, the columns remaining, and a note that the profile is simulated. Give the reason behind each choice in the user's own words where you have them. State every number with its unit. List the artifact path: `.workflow/artifacts/step2_profile.csv`.
