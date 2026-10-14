---
name: step-6-signoff
description: Step 6 of the model workflow. Summarises all prior decisions and warnings and collects the user's go/no-go decision, conditions and monitoring plan. Invoked by top-workflow; not for standalone use.
---

# Step 6: Sign-off

Human-driven step. Read `.workflow/state.json` in full.

## Do

1. Print a one-screen model card: target, data size after cleaning, features kept, model family, validation metric vs. the step 1 minimum, chosen cutoff with precision/recall/expected cost.
2. List every step whose status is `warn` or `overridden`, with the gate's findings or the override reason, so the user sees the open risks.
3. AskUserQuestion:
   - **Decision**: go / conditional go / no go. State plainly if the validation metric is below the step 1 minimum.
   - **Conditions** (if conditional, or if risks above need acknowledging): free text.
   - **Monitoring**: which to track after launch (multi-select: score drift, positive-rate drift, precision at cutoff, recall at cutoff, feature missingness) and the retrain/alert trigger as a short statement.

## Emit

End with exactly:

## DECISIONS
```json
{
  "step": 6,
  "decisions": {
    "decision": "go | conditional | no_go",
    "conditions": [],
    "acknowledged_risks": [],
    "monitoring_plan": {"metrics": [], "trigger": ""}
  },
  "interim_results": {
    "val_primary_metric": 0.0,
    "min_acceptable_metric": 0.0
  },
  "artifacts": []
}
```
