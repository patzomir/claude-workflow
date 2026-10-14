---
name: step-3-feature-selection
description: Step 3 of the model workflow. Simulates univariate information value and correlation screening on the cleaned columns and asks the user to set the cutoffs. Invoked by top-workflow; not for standalone use.
---

# Step 3: Feature selection

Simulation plus human decisions. Read `seed` from `.workflow/state.json` and the decision files `.workflow/decisions/step-1.md` and `step-2.md` (columns remaining, rows after cleaning).

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

End your reply with a `## DECISIONS` heading followed by a free-text markdown record and nothing after it. Write it for a reader who has seen none of the conversation (a person or another LLM): plain prose or bullets, no required schema. Include IV minimum, correlation cutoff, features selected, features dropped with a reason each, feature counts before/after, the maximum IV seen, and a note that the screening is simulated. Give the reason behind each choice in the user's own words where you have them. State every number with its unit. List the artifact path: `.workflow/artifacts/step3_screening.csv`.
