---
name: step-6-signoff
description: Step 6 of the model workflow. Summarises all prior decisions and warnings and collects the user's go/no-go decision, conditions and monitoring plan. Invoked by top-workflow; not for standalone use.
---

# Step 6: Sign-off

Human-driven step. Read `.workflow/state.json` (statuses, verdicts, override reasons) and every file in `.workflow/decisions/`.

## Do

1. Print a one-screen model card: target, data size after cleaning, features kept, model family, validation metric vs. the step 1 minimum, chosen cutoff with precision/recall/expected cost.
2. List every step whose status is `warn` or `overridden`, with the gate's findings or the override reason, so the user sees the open risks.
3. AskUserQuestion:
   - **Decision**: go / conditional go / no go. State plainly if the validation metric is below the step 1 minimum.
   - **Conditions** (if conditional, or if risks above need acknowledging): free text.
   - **Monitoring**: which to track after launch (multi-select: score drift, positive-rate drift, precision at cutoff, recall at cutoff, feature missingness) and the retrain/alert trigger as a short statement.

## Emit

End your reply with a `## DECISIONS` heading followed by a free-text markdown record and nothing after it. Write it for a reader who has seen none of the conversation (a person or another LLM): plain prose or bullets, no required schema. Include the decision (go / conditional go / no go), conditions, which risks from earlier warnings or overrides were acknowledged, the validation metric against the step 1 minimum, and the monitoring plan with its retrain/alert trigger. Give the reason behind each choice in the user's own words where you have them. State every number with its unit. No artifacts.
